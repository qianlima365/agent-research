---
title: Claude Code 基础架构
description: Claude Code 的 Orchestrator-Worker 分层隔离架构，包括四层分工、上下文隔离和权限模型。
tags: [claude-code, architecture, orchestrator, isolation]
---

# 基础架构

{% assign current_dir = page.path | remove: '/index.md' %}
{% for p in site.pages %}
  {% assign p_dir = p.path | split: '/' | pop | join: '/' %}
  {% if p_dir == current_dir and p.name != 'index.md' %}
- [{{ p.title }}]({{ p.url | relative_url }})
  {% endif %}
{% endfor %}
