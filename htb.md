---
title: Hack the Box Walkthroughs
layout: page
---

I've really enjoyed [Hack the Box](https://www.hackthebox.eu) as a learning environment for offensive security. Having to hack your way in to the site to sign up for an account is a nice touch. I'm still new to cybersecurity, so even the easier rated machine are a challenge, but it is rewarding when an exploit I'm trying finally works and uncovers a flag. I decided to start keeping track of my work and I'll be posting write ups here as the machines retire.

{% for walkthrough in site.htb %}

## [{{ walkthrough.title }}]({{ walkthrough.url | prepend: site.baseurl }})
  
  <!-- <h2>{{ walkthrough.title }}</h2>
  </a>
  -->
  {{ walkthrough.description | truncate: 160 }}
  
  {% endfor %}
