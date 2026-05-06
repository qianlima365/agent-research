---
title: 开发实践
---

# Agent 开发实践

## 概述

总结 Agent 开发中的工程化经验、设计模式与踩坑记录。

## 主题

<ul>
{% for page in site.pages %}
  {% if page.path contains 'development/' and page.name != 'index.md' %}
    <li><a href="{{ page.url | relative_url }}">{{ page.title }}</a></li>
  {% endif %}
{% endfor %}
</ul>

## 核心议题

- Prompt Engineering 与指令设计
- 工具定义（Tool Definition）规范
- Agent 状态机与生命周期管理
- 错误处理与降级策略
- 可观测性（Tracing / Logging / Metrics）
- 测试策略（确定性验证、模拟环境）
