---
title: HTB Walkthroughs
layout: page
---

Testing which page is loading. This is the one in the collection directory.

{% for walkthrough in site.htb %}

<a href="{{ walkthrough.url | prepend: site.baseurl }}">
  <h2>{{ htb.title }}</h2>
  </a>
  
  <p class ="post-excerpt">{{ walkthrough.description | truncate: 160 }}</p>
  
  {% endfor %}
