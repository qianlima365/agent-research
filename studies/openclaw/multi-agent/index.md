---
title: OpenClaw 多智能体设计
description: OpenClaw 的去中心化多智能体协作模型，包括 agentToAgent 原语、网状通信和动态组队。
tags: [openclaw, multi-agent, decentralized, collaboration]
---

# OpenClaw 多智能体设计

{% assign current_dir = page.path | remove: '/index.md' %}
{% for p in site.pages %}
  {% assign p_dir = p.path | split: '/' | pop | join: '/' %}
  {% if p_dir == current_dir and p.name != 'index.md' %}
- [{{ p.title }}]({{ p.url | relative_url }})
  {% endif %}
{% endfor %}
