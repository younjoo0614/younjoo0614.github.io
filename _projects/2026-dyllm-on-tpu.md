---
title: "DyLLM on TPU"
date: 2026-04-28
excerpt: "Running and optimizing DyLLM on Google TPU hardware."
---

# Porting DyLLM to TPU: Making Dynamic Sparse Inference XLA-Friendly

The original code is available [here](https://github.com/younjoo0614/DyLLM_tpu.git).

DyLLM is a training-free inference framework for diffusion language models. The key observation is that, during iterative denoising, not every token changes significantly at every step. DyLLM exploits this by selecting salient tokens after attention, recomputing expensive FFN operations mainly for those tokens, and reusing cached activations for the rest. More details are explained in our paper: [DyLLM](https://arxiv.org/abs/2603.08026).

The original DyLLM implementation is GPU-oriented: sparse attention and cache kernels can be accelerated with custom CUDA kernels. Porting this idea to TPU requires a different style of optimization. TPU execution through PyTorch/XLA is compiler-driven, so the goal is not only to port the functionality of DyLLM, but also to express DyLLM’s dynamic sparse computation in a way that is friendly to XLA compilation and distributed execution.

## Parallelism

I implemented DyLLM on a TPU v5e pod. Since each TPU v5e chip has a limited amount of HBM memory, parallelism is inevitable for running large diffusion language models such as LLaDA 8B Instruct and Dream 7B Instruct.

In the TPU implementation, parallelism is important for two reasons. First, the model weights and intermediate activations may not fit comfortably on a single chip. Second, DyLLM’s saliency decision is based on attention context similarity, which means that even when computation is partitioned across chips, the saliency metadata must remain consistent across ranks.

This makes multi-chip communication part of the inference pipeline. In particular, DyLLM needs collective operations during the attention stage to compute global cosine similarity for salient-token selection. Even with a Megatron-like tensor-parallel implementation, this communication needs to be handled carefully because saliency selection affects which tokens are recomputed in the next step.

## Padding for Salient Tokens

One of the most important TPU-specific optimizations is padding the salient-token index tensor.

In DyLLM, salient tokens are selected layer by layer based on how much their attention context changes. Therefore, the number of salient tokens is dynamic. Without padding, the shape of the salient-token index tensor may change at every layer and every denoising step.

For example, the index tensor may have shapes like:

```text
[13]
[27]
[91]
[118]
```

XLA specializes compiled graphs to tensor shapes, so each previously unseen shape may trigger recompilation. This does not necessarily mean recompilation happens at every step, but dynamic sparse tensor shapes can cause frequent recompilation in practice.

To reduce this overhead, I bucket the salient-token index tensor. The implementation rounds the count up to the next power of two, with a minimum bucket size of 32, and pads with the last valid index within each sequence.

```python
bucket = 1 << max(5, (cnt - 1).bit_length())
if bucket > seg_len:
    bucket = seg_len
```

This transforms many possible dynamic shapes into a smaller set of reusable shapes:

```text
1–32 salient tokens     -> 32
33–64 salient tokens    -> 64
65–128 salient tokens   -> 128
129–256 salient tokens  -> 256
```

Padding does not eliminate recompilation entirely, but it significantly reduces the number of distinct sparse shapes XLA sees. This makes DyLLM’s dynamic sparsity much more compatible with PyTorch/XLA’s compilation model.

## `F.scaled_dot_product_attention` Compatibility

Fortunately, the TPU implementation can keep attention expressed in the standard scaled dot-product attention form. While PyTorch’s CUDA backend may dispatch `F.scaled_dot_product_attention` to FlashAttention-2 or memory-efficient kernels, the TPU/XLA path relies on the general attention formulation lowered through PyTorch/XLA.

The benefit is portability. The code avoids depending on CUDA-specific attention kernels while still exposing a standard attention pattern that future XLA/libtpu optimizations may be able to improve.

Conceptually, the attention computation remains:

```python
scores = torch.matmul(q, k.transpose(-1, -2)) * scale
scores = scores + attention_bias_or_mask
probs = torch.softmax(scores, dim=-1)
out = torch.matmul(probs, v)
```

This should not be interpreted as “FlashAttention on TPU.” Instead, the advantage is that attention is written in a backend-portable form. The TPU compiler stack can lower and optimize this standard computation pattern, and future PyTorch/XLA or libtpu improvements may further improve performance without requiring a CUDA-specific attention path.

## Cache Pre-allocation to Avoid XLA Graph Changes

Another important optimization is cache pre-allocation.

DyLLM uses caches for attention context and value tensors. If cache allocation happens inside the forward pass through dynamically shaped operations such as `torch.cat`, the XLA graph can change when new sequences arrive or sequence lengths change. This may trigger recompilation.

To avoid this, cache allocation is moved outside the model forward path. In the runner, cache pre-allocation happens before the model execution:

```python
def run(self, seqs: list[Sequence], is_full: bool) -> list[int]:
    input_ids, positions = self.prepare_full(seqs) if is_full else self.prepare_sparse(seqs)

    self._pre_allocate_caches(is_full)

    if self.device.type == "xla":
        import torch_xla
        torch_xla.sync()

    logits = self.run_model(input_ids, positions, is_full)
```

This separates cache allocation from cache access:

```text
Before forward:
    allocate cache storage for active sequences

During forward:
    only read/write existing cache tensors
```

As a result, the forward path stays closer to a stable tensor program. This is important for TPU execution because PyTorch/XLA performs best when the traced graph structure and tensor shapes remain stable across iterations.

## XLA-Friendly Cache Operations

DyLLM’s cache is not a simple batch-major tensor. It is a global storage region shared by active sequences. Each active sequence owns a contiguous region in the cache, and the cache manager tracks where each sequence starts.

Conceptually, token `j` of sequence `s` is stored at:

```text
cache_row = seq_starts[s] + j
```

The challenge is that the model input is packed across multiple sequences. A row in the packed input does not directly tell us which cache row it corresponds to. The TPU implementation therefore computes this mapping using tensor operations such as `torch.arange`, `torch.searchsorted`, `index_select`, and `index_copy`.

A full cache read can be described as:

```python
rows = torch.arange(total_rows, device=device)
seg = torch.searchsorted(cu_seqlens[1:], rows, right=True)

seq_id_per_row = seq_ids.index_select(0, seg)
local = rows - cu_seqlens.index_select(0, seg)

cache_row = seq_starts.index_select(0, seq_id_per_row) + local
out = cache.index_select(0, cache_row)
```

A full cache reset uses the same mapping, but writes the new values back:

```python
cache = cache.index_copy(0, cache_row, src)
```

This avoids Python-side per-token cache loops inside the forward path. Instead, cache reads and writes are represented as batched tensor programs that PyTorch/XLA can trace.

The implementation also supports response-block cache operations. In sparse steps, the query side may contain only response tokens, while the key/value side still corresponds to the full sequence. In that case, the query-local row must be mapped to the response block inside the full sequence:

```text
cache_row = seq_start + (k_len - q_len) + local_q
```

This allows DyLLM to update only the response part of the cache during response-focused sparse decoding while preserving the correct full-sequence cache layout.

## Results

End-to-end acceleration is tested on the LLaDA 8B Instruct and Dream 7B Instruct models. On GSM8K sequences, average inference time is reduced from **520.42s to 354.32s** with LLaDA and from **456.76s to 318.63s** with Dream at batch size 1.

```text
LLaDA 8B Instruct:
    Original: 520.42s
    DyLLM:    354.32s
    Speedup:  ~32%

Dream 7B Instruct:
    Original: 456.76s
    DyLLM:    318.63s
    Speedup:  ~30%
```

Most sparse steps show significant speedup, often around 2x. However, occasional recompilation steps still limit the overall advantage of DyLLM on TPU.

## Future Work

There are several directions for further optimization.

### 1. Better Heuristics for Dynamic Tensor Shapes

The current salient-token padding strategy reduces recompilation by bucketing dynamic index tensors. However, dynamic shapes are still a major challenge. Better heuristics could further stabilize sparse tensor shapes across layers and denoising steps.

### 2. Fusion for Sparse Attention Operations

Sparse attention operations can be further fused. In the current implementation, attention computation, context update, and cosine similarity measurement are separate operations. Fusing attention and cosine similarity measurement could improve memory utilization and reduce intermediate tensor materialization.

### 3. Multi-chip Communication

DyLLM requires collective communication during the attention stage, even with Megatron-like tensor parallelism. This is because saliency selection depends on global context similarity, and all ranks must agree on which tokens are salient.

The current implementation uses a conservative communication path, but multi-chip communication remains an important optimization target. Better collective support or native XLA-based communication could reduce synchronization overhead and improve scalability.

## Takeaway

Porting DyLLM to TPU is not just a matter of replacing CUDA kernels. DyLLM’s sparse inference pattern is highly dynamic, while PyTorch/XLA performs best with stable, traceable tensor programs. The TPU implementation therefore focuses on making dynamic sparsity XLA-friendly through salient-token padding, cache pre-allocation, tensorized cache operations, and backend-portable attention formulation.

The key lesson is that TPU optimization is often about controlling dynamism: keeping shapes stable, avoiding allocation inside the forward path, and expressing sparse operations in a form that the compiler can understand.
