---
title: HTB Walkthroughs
---

Testing which page is loading, this is the one in the root directory.

{% for walkthrough in site.htb %}

<a href="{{ walkthrough.url | prepend: site.baseurl }}">
  <h2>{{ walkthrough.title }}</h2>
  </a>
  
  <p class ="post-excerpt">{{ walkthrough.description | truncate: 160 }}</p>
  
  {% endfor %}
