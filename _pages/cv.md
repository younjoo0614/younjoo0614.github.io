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

- Add service entries here.
