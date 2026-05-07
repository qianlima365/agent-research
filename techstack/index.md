---
title: 技术栈
---

## 概述

梳理构建 AI Agent 所需的基础设施、中间件与工具链，辅助技术选型决策。

## 内容

<ul>
{% for page in site.pages %}
  {% if page.path contains 'techstack/' and page.name != 'index.md' %}
    <li><a href="{{ page.url | relative_url }}">{{ page.title }}</a></li>
  {% endif %}
{% endfor %}
</ul>

## 技术分层

| 层级 | 组件 |
|------|------|
| Model Layer | OpenAI, Anthropic, 开源模型, vLLM |
| Orchestration | LangChain, LlamaIndex, Semantic Kernel |
| Memory | Vector DB (Pinecone, Milvus, Chroma), Redis |
| Tools | Search, Code Execution, Browser Automation |
| Deployment | FastAPI, Docker, K8s, Serverless |
| Observability | LangSmith, Langfuse, OpenTelemetry |
