---
title: Claude Code Ultra Plan
description: 三探索 + 一评审的并行探索机制，模拟人类团队的头脑风暴 + 独立评审流程。
tags: [claude-code, ultra-plan, multi-agent, parallel]
---

# Claude Code Ultra Plan

{% assign current_dir = page.path | remove: '/index.md' %}
{% for doc in site.studies %}
  {% assign doc_dir = doc.path | split: '/' | pop | join: '/' %}
  {% if doc_dir == current_dir and doc.name != 'index.md' %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
  {% endif %}
{% endfor %}
