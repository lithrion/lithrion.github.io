---
title: Hack the Box Walkthroughs
layout: page
---

I've really enjoyed [Hack the Box](https://www.hackthebox.eu) as a learning environment for offensive security. From the very beginning, having to hack your way in to sign up for an account is a nice touch. I'm still new to cybersecurity, so even the easier rated machine are definately a challenge to get into, but it is pretty rewards when one of the thigns I'm trying finally works and uncovers a flag. I decided to start keeping track of my work and I'll be posting write ups here as the machines retire.

{% for walkthrough in site.htb %}

## [{{ walkthrough.title }}]({{ walkthrough.url | prepend: site.baseurl }})
  
  <!-- <h2>{{ walkthrough.title }}</h2>
  </a>
  -->
  {{ walkthrough.description | truncate: 160 }}
  
  {% endfor %}
