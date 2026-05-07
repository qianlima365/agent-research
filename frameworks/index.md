---
title: 框架
---

## 概述

对比分析主流 AI Agent 开发框架的设计理念、适用场景与生态成熟度。

## 收录框架

<div class="project-list">
{% for page in site.pages %}
  {% if page.path contains 'frameworks/' and page.name != 'index.md' %}
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

## 对比维度

- 编排能力（Workflow vs Autonomous）
- 多 Agent 协作支持
- 记忆机制（短期 / 长期 / 向量检索）
- 工具调用与扩展性
- 生产就绪程度
