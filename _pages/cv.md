---
layout: cv
title: "Curriculum Vitae"
description: "Education, work experience, publications, talks and teaching."
permalink: /cv/
---

## Education

- **Ph.D.** &mdash; *Your University*, year (in progress / expected year)
- **M.S.** &mdash; *Your University*, year
- **B.S.** &mdash; *Your University*, year

## Work experience

- **Year &ndash; Year**: *Position*
  - *Affiliation*
  - Brief description of duties / responsibilities.

- **Year &ndash; Year**: *Position*
  - *Affiliation*
  - Brief description of duties / responsibilities.

## Skills

- Skill 1
- Skill 2
  - Sub-skill 2.1
  - Sub-skill 2.2
- Skill 3

## Publications

<ul>
{% assign pubs = site.publications | sort: 'date' | reverse %}
{% for post in pubs %}
	<li>
		<a href="{{ post.url | relative_url }}">{{ post.title }}</a>
		{% if post.venue %} &mdash; <em>{{ post.venue }}</em>{% endif %}
		{% if post.date %} ({{ post.date | date: "%Y" }}){% endif %}
	</li>
{% endfor %}
</ul>

## Talks

<ul>
{% assign talks = site.talks | sort: 'date' | reverse %}
{% for post in talks %}
	<li>
		<a href="{{ post.url | relative_url }}">{{ post.title }}</a>
		{% if post.venue %} &mdash; {{ post.venue }}{% endif %}
		{% if post.date %} ({{ post.date | date: "%b %Y" }}){% endif %}
	</li>
{% endfor %}
</ul>

## Teaching

<ul>
{% assign teach = site.teaching | sort: 'date' | reverse %}
{% for post in teach %}
	<li>
		<a href="{{ post.url | relative_url }}">{{ post.title }}</a>
		{% if post.venue %} &mdash; {{ post.venue }}{% endif %}
		{% if post.date %} ({{ post.date | date: "%Y" }}){% endif %}
	</li>
{% endfor %}
</ul>

## Service and leadership

- Add service entries here.
