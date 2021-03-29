---
title: UCSD Cybersecurity Bootcamp Projects
layout: page
---
# Class Projects

This page contains some of the projects that I completed as part of a Cybersecurity Bootcamp offered through UCSD extension. The course covered a wide range of Cybersecurity fundamentals, and the projects highlight a few of the major topics that were covered.

{% for project in site.class-projects %}

[{{ project.title }}]({{ project.url | prepend: site.baseurl }})
{{ project.description | truncate: 160 }}

{% endfor %}
