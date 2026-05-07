---
title: 基础架构
description: OpenClaw 的注册中心 + 消息总线 + 多平台 Gateway 三层架构详解。
tags: [openclaw, architecture, gateway, registry]
---

{% assign current_dir = page.path | remove: '/index.md' %}
{% for p in site.pages %}
  {% assign p_dir = p.path | split: '/' | pop | join: '/' %}
  {% if p_dir == current_dir and p.name != 'index.md' %}
- [{{ p.title }}]({{ p.url | relative_url }})
  {% endif %}
{% endfor %}
