---
title: 开源项目
---

# 开源 Agent 项目

## 概述

收集并跟踪 AI Agent 领域高质量开源项目，涵盖通用 Agent、垂直领域 Agent、Multi-Agent 系统等方向。

## 项目列表

<ul>
{% for page in site.pages %}
  {% if page.path contains 'projects/' and page.name != 'index.md' %}
    <li><a href="{{ page.url | relative_url }}">{{ page.title }}</a></li>
  {% endif %}
{% endfor %}
</ul>

## 分类标签

- `general` — 通用型 Agent
- `coding` — 编程辅助 Agent
- `research` — 科研/信息收集 Agent
- `multi-agent` — 多 Agent 协作系统
- `autonomous` — 自主执行型 Agent
