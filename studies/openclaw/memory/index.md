---
title: OpenClaw 记忆层设计
description: OpenClaw 的长期记忆、状态持久化和 Cron 常驻运行机制。
tags: [openclaw, memory, persistence, cron]
---

# OpenClaw 记忆层设计

{% assign current_dir = page.path | remove: '/index.md' %}
{% for doc in site.studies %}
  {% assign doc_parts = doc.path | split: '/' %}
  {% assign doc_dir = doc.path | split: '/' | pop | join: '/' %}
  {% if doc_dir == current_dir and doc_parts.size == 4 %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
  {% endif %}
{% endfor %}
