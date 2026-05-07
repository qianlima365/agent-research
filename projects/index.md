---
title: 开源项目
---

## 概述

收集并跟踪 AI Agent 领域高质量开源项目，涵盖通用 Agent、垂直领域 Agent、Multi-Agent 系统等方向。

## 项目列表

<div class="project-list">
{% for page in site.pages %}
  {% if page.path contains 'projects/' and page.name != 'index.md' %}
    <a href="{{ page.url | relative_url }}" class="project-card">
      <h3 class="project-title">{{ page.title }}</h3>
      {% if page.description %}
        <p class="project-desc">{{ page.description }}</p>
      {% endif %}
      {% if page.tags %}
        <div class="project-tags">
          {% for tag in page.tags %}
            <span class="project-tag">{{ tag }}</span>
          {% endfor %}
        </div>
      {% endif %}
    </a>
  {% endif %}
{% endfor %}
</div>

## 分类标签

- `general` — 通用型 Agent
- `coding` — 编程辅助 Agent
- `research` — 科研/信息收集 Agent
- `multi-agent` — 多 Agent 协作系统
- `autonomous` — 自主执行型 Agent
