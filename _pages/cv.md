---
layout: cv
title: "Curriculum Vitae"
description: "Education, publications, experience, talks and skills."
permalink: /cv/
---

## Education

- **Sep. 2024 &ndash; Present** &mdash; *Seoul National University (SNU)*, M.S./Ph.D. in Intelligence and Information
	- Advisor: Prof. Jung Ho Ahn (SCALE lab)
- **Sep. 2022 &ndash; Feb. 2023** &mdash; Research intern at [SAFARI](https://safari.ethz.ch), *ETH Zürich*
- **Feb. 2022 &ndash; Aug. 2022** &mdash; Exchange program, *ETH Zürich*
- **Feb. 2019 &ndash; Aug. 2024** &mdash; *Seoul National University (SNU)*, B.S. in Electrical and Computer Engineering

## Publications

<ul>
{% assign pubs = site.publications | sort: 'date' | reverse %}
{% for post in pubs %}
	<li>
		{% assign authors_display = post.authors | default: '' | replace: 'Younjoo Lee', '<strong>Younjoo Lee</strong>' %}
		{% if authors_display != '' %}{{ authors_display }}. {% endif %}<em>{{ post.title }}</em>.{% if post.venue %} {{ post.venue }}.{% endif %}{% if post.date %} {{ post.date | date: "%Y" }}{% endif %}
		{% if post.excerpt and post.excerpt contains 'Nominated for Best Paper' %}
			<br />- <em>Nominated for Best Paper</em>
		{% endif %}
	</li>
{% endfor %}
</ul>

## Experience

- **Google TPU Builder Program** &mdash; Apr. 2026 &ndash; Present
	- Built efficient diffusion LLM framework (DyLLM) on TPU using PyTorch XLA and Pallas kernels in TPU Sprint.
	<!-- - Attended Google AI DevLabs 2026 (Sunnyvale, California) with DyLLM project. -->
- **GPU/HPC Server Administration**, SCALE lab &mdash; Sep. 2024 &ndash; Present
	- Plan and configure GPU/HPC server specifications based on workload requirements, including motherboard, PSU, GPU, CPU, memory, storage, cooling, etc.
	- Troubleshoot server, software, environment, and network issues.
	- Set up and maintain servers.
- **Safari research internship**, ETH Zürich &mdash; Sep. 2022 &ndash; Feb. 2023
	- Project name: 'Comprehensive characterization and optimization of seeding algorithms on modern FPGAs'
	- Implemented and accelerated various seeding algorithms on a full read-mapping FPGA/CPU pipeline and evaluated their accuracy, performance, etc. on 3 major read types (ONT, HiFi, Illumina).
	- Developed HLS kernels, tested on a real Alveo U55C FPGA provided through the Xilinx HACC program.
	- Teacher of Project & Seminar course at ETH: Sep. 2022 &ndash; Dec. 2022.
- **LikeLion 8th (2020), Wafflestudio (2021)** &mdash; web service engineering clubs in SNU
	- Launched a social network web service for tracking users' reading activity on their own online bookshelf using the Django framework on AWS Elastic Beanstalk.

## Academic Services

- Reviewer, IEEE Transactions on Computers (TC), 2026
- Artifact Evaluation Reviewer, HPCA, 2026

## Skills & Languages

- CUDA/C++, PyTorch/XLA, FPGA HLS, GPU/HPC server maintenance
- Languages: Korean (native), English (fluent)

## Projects

<ul>
{% assign projs = site.projects | sort: 'date' | reverse %}
{% for post in projs %}
	<li>
		{% if post.external_url %}
			<a href="{{ post.external_url }}" target="_blank" rel="noopener">{{ post.title }}</a>
		{% else %}
			<a href="{{ post.url | relative_url }}">{{ post.title }}</a>
		{% endif %}
		{% if post.date %} ({{ post.date | date: "%Y" }}){% endif %}
	</li>
{% endfor %}
</ul>

## Leadership and Awards

- Cheongpa Scholarship (2025, 2026)
- Seoul National University varsity Tennis Team
- President, SNU Engineering Tennis Club
	- Held several tennis tournaments in SNU.

