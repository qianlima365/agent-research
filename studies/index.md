---
title: 专题研究
---

# 专题列表

{% for page in site.pages %}
  {% assign path_parts = page.path | split: '/' %}
  {% if path_parts[0] == 'studies' and path_parts.size == 3 and page.name == 'index.md' %}
- [{{ page.title }}]({{ page.url | relative_url }})
  {% endif %}
{% endfor %}
