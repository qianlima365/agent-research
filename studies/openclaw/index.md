---
title: OpenClaw 专题
permalink: /studies/openclaw/
description: 开源多通道 AI Agent 框架，通过统一 Gateway 接入 20+ 通信平台，让 Agent 成为真正的数字员工。
tags: [openclaw, multi-agent, cli, open-source]
---

# OpenClaw 专题

{% assign path_parts = page.path | split: '/' %}
{% assign topic = path_parts[1] %}
{% for doc in site.studies %}
  {% assign doc_parts = doc.path | split: '/' %}
  {% if doc_parts[1] == topic and doc_parts.size == 4 and doc.path != page.path %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
  {% endif %}
{% endfor %}
