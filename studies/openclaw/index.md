---
title: OpenClaw 专题
permalink: /studies/openclaw/
description: 开源多通道 AI Agent 框架，通过统一 Gateway 接入 20+ 通信平台，让 Agent 成为真正的数字员工。
tags: [openclaw, multi-agent, cli, open-source]
---

# OpenClaw 专题

## 调试：site.studies 所有文档

| path | title | name |
|------|-------|------|
{% for doc in site.studies %}| `{{ doc.path }}` | {{ doc.title }} | `{{ doc.name }}` |
{% endfor %}

## 调试：当前页面信息

- page.path: `{{ page.path }}`
- page.name: `{{ page.name }}`
- page.title: `{{ page.title }}`

## 调试：筛选结果

{% assign path_parts = page.path | split: '/' %}
- topic (path_parts[0]): `{{ path_parts[0] }}`
- topic (path_parts[1]): `{{ path_parts[1] }}`

{% for doc in site.studies %}
  {% assign doc_parts = doc.path | split: '/' %}
  {% assign doc_filename = doc.path | split: '/' | last %}
- doc: `{{ doc.path }}` | parts[0]=`{{ doc_parts[0] }}` | parts[1]=`{{ doc_parts[1] }}` | size={{ doc_parts.size }} | filename=`{{ doc_filename }}`
{% endfor %}

## 研究方向列表

{% for doc in site.studies %}
  {% assign doc_parts = doc.path | split: '/' %}
  {% assign doc_filename = doc.path | split: '/' | last %}
  {% if doc_parts[0] == 'openclaw' and doc_parts.size == 3 and doc_filename == 'index.md' and doc.path != page.path %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
  {% endif %}
{% endfor %}
