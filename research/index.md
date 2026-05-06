---
title: 研究笔记
---

# 研究笔记

## 概述

记录对 Agent 领域论文、趋势与关键问题的思考。

## 笔记列表

<ul>
{% for item in site.research %}
  {% if item.title != "研究笔记" %}
    <li><a href="{{ item.url | relative_url }}">{{ item.title }}</a></li>
  {% endif %}
{% endfor %}
</ul>

## 关注方向

- Multi-Agent 协作与博弈
- Agent 安全与对齐
- 规划（Planning）与推理能力边界
- 长期记忆与自我进化
- Agent 评估基准与方法
