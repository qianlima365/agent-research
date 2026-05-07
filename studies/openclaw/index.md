---
title: OpenClaw 专题
permalink: /studies/openclaw/
description: 开源多通道 AI Agent 框架，通过统一 Gateway 接入 20+ 通信平台，让 Agent 成为真正的数字员工。
tags: [openclaw, multi-agent, cli, open-source]
---

# 专题方向

{% assign topic = page.path | split: '/' | slice: 1 | first %}
{% for p in site.pages %}
  {% assign parts = p.path | split: '/' %}
  {% if parts[0] == 'studies' and parts[1] == topic and parts.size == 4 and p.name == 'index.md' and p.path != page.path %}
- [{{ p.title }}]({{ p.url | relative_url }})
  {% endif %}
{% endfor %}
