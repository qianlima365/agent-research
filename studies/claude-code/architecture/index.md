---
title: Claude Code 基础架构
description: Claude Code 的 Orchestrator-Worker 分层隔离架构，包括四层分工、上下文隔离和权限模型。
tags: [claude-code, architecture, orchestrator, isolation]
---

# Claude Code 基础架构

{% assign current_dir = page.path | remove: '/index.md' %}
{% for doc in site.studies %}
  {% assign doc_parts = doc.path | split: '/' %}
  {% assign doc_dir = doc.path | split: '/' | pop | join: '/' %}
  {% if doc_dir == current_dir and doc_parts.size == 4 %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
  {% endif %}
{% endfor %}
