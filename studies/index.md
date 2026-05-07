---
title: 专题研究
---

# 专题研究

## 概述

对 AI Agent 领域最热门的产品、开源项目和技术栈进行深度专题研究。每个专题从架构设计、核心原理、生态发展到实践落地进行系统性拆解。

## 进行中的专题

<div class="project-list">
{% for page in site.pages %}
  {% assign path_parts = page.path | split: '/' %}
  {% if path_parts[0] == 'studies' and path_parts.size == 2 and page.name == 'index' %}
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

## 专题规划方向

- **OpenClaw** — 多通道 AI Agent 框架的架构与生态
- **Claude Code** — Anthropic 官方 CLI 的多智能体设计哲学
- **Agency Orchestrator** — DAG 工作流编排引擎的工程实践
- **LangGraph** — 状态机驱动的复杂工作流编排
- **AutoGen** — 对话即编程的多智能体协作模式
