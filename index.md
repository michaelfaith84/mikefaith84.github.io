---
layout: default
title: Home
---

{% assign latest = site.posts.first %}
{% if latest %}
<div class="latest-post">
  <h2><a href="{{ latest.url | relative_url }}">{{ latest.title | escape }}</a></h2>
  <p class="post-meta">{{ latest.date | date: "%B %-d, %Y" }}</p>
  {{ latest.content }}
</div>
{% endif %}

{% include tag-cloud.html %}
