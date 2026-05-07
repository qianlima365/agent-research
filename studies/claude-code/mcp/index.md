---
title: Claude Code MCP 协议
description: Model Context Protocol 详解、生态现状和安全模型。
tags: [claude-code, mcp, protocol, tools]
---

# MCP 协议

{% assign current_dir = page.path | remove: '/index.md' %}
{% for p in site.pages %}
  {% assign p_dir = p.path | split: '/' | pop | join: '/' %}
  {% if p_dir == current_dir and p.name != 'index.md' %}
- [{{ p.title }}]({{ p.url | relative_url }})
  {% endif %}
{% endfor %}
