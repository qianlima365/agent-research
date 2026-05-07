---
title: OpenClaw 基础架构
description: OpenClaw 的注册中心 + 消息总线 + 多平台 Gateway 三层架构详解。
tags: [openclaw, architecture, gateway, registry]
---

# OpenClaw 基础架构

{% assign current_dir = page.path | remove: '/index.md' %}
{% for doc in site.studies %}
  {% assign doc_parts = doc.path | split: '/' %}
  {% assign doc_dir = doc.path | split: '/' | pop | join: '/' %}
  {% if doc_dir == current_dir and doc_parts.size == 4 %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
  {% endif %}
{% endfor %}
