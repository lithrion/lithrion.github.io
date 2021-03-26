---
title: UCSD Cybersecurity Bootcamp Projects
---
# Class Projects
- ELK Stack Project
- Final Project

This page contains some of the projects that I completed as part of a Cybersecurity Bootcamp offered through UCSD extension. The course covered a wide range of Cybersecurity fundamentals, and the projects highlight a few of the major topics that were covered.

{% for project in site.class-projects %}

[{{ project.title }}]({{ walkthrough.url | prepend: site.baseurl }})
{{ walkthrough.description | truncate: 160 }}

{% endfor %}
