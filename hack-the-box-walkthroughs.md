{% for walkthrough in site.hack-the-box-walkthroughs %}

<a href="{{ walkthroughs.url | prepend: site.baseurl }}">
  <h2>{{ themes.title }}</h2>
  </a>
  
  <p class ="post-excerpt">{{ walkthrough.description | truncate: 160 }}</p>
  
  {% endfor %}
