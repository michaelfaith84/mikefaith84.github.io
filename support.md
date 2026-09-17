---
layout: default
title: Support
permalink: /support/
---

If you appreciate what I do, please consider supporting me in some way. I hate paywalls so unless there is an overwhelming demand, there will be no exclusive content.

<ul class="support-list">
{% for link in site.support %}
  {% if link.url and link.url != "" %}
  <li>
    <a class="support-link" href="{{ link.url }}" rel="noopener">{{ link.label }}</a>
    <span class="support-note">{{ link.note }}</span>
  </li>
  {% endif %}
{% endfor %}
</ul>
