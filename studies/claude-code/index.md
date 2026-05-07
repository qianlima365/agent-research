---
title: Claude Code 专题
permalink: /studies/claude-code/
description: Anthropic 官方 CLI 工具，深度集成于开发者工作流，以代码感知、MCP 生态和分层隔离架构著称。
tags: [claude-code, anthropic, mcp, multi-agent]
---

# Claude Code 专题

{% assign topic = page.path | split: '/' | slice: 1 | first %}
{% for p in site.pages %}
  {% assign parts = p.path | split: '/' %}
  {% if parts[0] == 'studies' and parts[1] == topic and parts.size == 4 and p.name == 'index.md' and p.path != page.path %}
- [{{ p.title }}]({{ p.url | relative_url }})
  {% endif %}
{% endfor %}
