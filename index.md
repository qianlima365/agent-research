---
layout: home
title: 首页
---

# AI Agent Research

聚焦 AI Agent 领域，收录开源项目、开发框架、技术栈与实践经验。

## 内容板块

| 板块 | 说明 |
|------|------|
| [框架](frameworks/) | Agent 开发框架对比与深入分析 |
| [开源项目](projects/) | 值得关注的开源 Agent 项目 |
| [开发实践](development/) | 模式、最佳实践与工程化经验 |
| [技术栈](techstack/) | LLM、工具链、基础设施选型 |
| [研究笔记](research/) | 论文解读、趋势观察与思考 |
| [专题研究](studies/) | 热门产品/开源项目/技术栈深度专题 |

## 最近更新

<ul>
  {% for page in site.pages %}
    {% if page.date %}
      <li><a href="{{ page.url | relative_url }}">{{ page.title }}</a> — {{ page.date | date: "%Y-%m-%d" }}</li>
    {% endif %}
  {% endfor %}
</ul>
