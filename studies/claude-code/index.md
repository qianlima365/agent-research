---
title: Claude Code 专题
permalink: /studies/claude-code/
description: Anthropic 官方 CLI 工具，深度集成于开发者工作流，以代码感知、MCP 生态和分层隔离架构著称。
tags: [claude-code, anthropic, mcp, multi-agent]
---

# Claude Code 专题

{% assign path_parts = page.path | split: '/' %}
{% assign topic = path_parts[1] %}
{% for doc in site.studies %}
  {% assign doc_parts = doc.path | split: '/' %}
  {% if doc_parts[1] == topic and doc_parts.size == 4 and doc.path != page.path %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
  {% endif %}
{% endfor %}
