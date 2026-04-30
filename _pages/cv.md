---
layout: cv
title: "Curriculum Vitae"
description: "Education, publications, experience, talks and skills."
permalink: /cv/
---

## Education

- **Present** *Seoul National University*, Graduate School of Convergence Science and Technology
- **B.S.** &mdash; *Seoul National University*, Electrical and Computer engineering 2024
- **Exchange program & Research internship at [SAFARI](https://safari.ethz.ch)** *ETH Zürich* 2022

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

- Google TPU Builder Apr. 2026 - Now
	- Built efficient diffusion LLM framework (DyLLM) on TPU using PyTorch and Pallas kernels
	<!-- - Attended Google AI DevLabs 2026 (Sunnyvale, California) with DyLLM project. -->
- Safari research internship: Sep. 2022 - Dec. 2023
	- Project name: 'Comprehensive characterization and optimization of seeding algorithms on modern FPGAs'
	- Implemented various seeding algorithms on full read mapping FPGA/CPU pipeline and evaluated their accuracy, performance etc. on 3 major read types (ONT, HiFi, Illumina)
	- HLS kernels, testing on real FPGA(alveo-u55c) privileged by HACC program of Xilinx.
	- T/A of Project & Seminar course in ETH: Sep. 2022 - Dec. 2022
- LikeLion 8th (2020), Wafflestudio (2021) both web service engineering club in SNU
	- Launched social network web service logging what people read on their own online bookshelf using Django framework on AWS Elastic Beanstalk.



## Skills & Languages

- PyTorch, CUDA programming, GPU system maintenance
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

## Service and leadership

- Seoul National University varsity Tennis Team
- Presiduent of SNU tennis club in Engineering Department

