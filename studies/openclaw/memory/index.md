---
title: 记忆层设计
description: OpenClaw 的长期记忆、状态持久化和 Cron 常驻运行机制。
tags: [openclaw, memory, persistence, cron]
---

{% assign current_dir = page.path | remove: '/index.md' %}
{% for p in site.pages %}
  {% assign p_dir = p.path | split: '/' | pop | join: '/' %}
  {% if p_dir == current_dir and p.name != 'index.md' %}
- [{{ p.title }}]({{ p.url | relative_url }})
  {% endif %}
{% endfor %}
