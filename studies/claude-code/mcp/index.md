---
title: Claude Code MCP 协议
description: Model Context Protocol 详解、生态现状和安全模型。
tags: [claude-code, mcp, protocol, tools]
---

# Claude Code MCP 协议

{% assign current_dir = page.path | remove: '/index.md' %}
{% for doc in site.studies %}
  {% assign doc_parts = doc.path | split: '/' %}
  {% assign doc_dir = doc.path | split: '/' | pop | join: '/' %}
  {% if doc_dir == current_dir and doc_parts.size == 4 %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
  {% endif %}
{% endfor %}
