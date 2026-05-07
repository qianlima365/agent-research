---
title: Claude Code 专题
permalink: /studies/claude-code/
description: Anthropic 官方 CLI 工具，深度集成于开发者工作流，以代码感知、MCP 生态和分层隔离架构著称。
tags: [claude-code, anthropic, mcp, multi-agent]
---

# Claude Code 专题

**开发者**: Anthropic（官方） | **协议**: 闭源（免费使用）

> 从 IDE 内向外的深度集成——不是又一个聊天窗口，而是理解你代码库的工程伙伴。

---

## 产品定位

Claude Code 是 Anthropic 推出的官方 CLI 工具，核心设计理念是**"深度集成"**——它不是独立运行的聊天机器人，而是嵌入在开发者工作流中的**代码感知型 AI 助手**。它能读取你的代码库、理解项目结构、执行命令、操作文件，并且支持多 Agent 协作。

与 OpenClaw 的"水平广度"不同，Claude Code 追求"垂直深度"：把代码开发这个场景做到极致，从代码生成到审查到重构，形成闭环。

---

## Agent 定义

Claude Code 的 Agent 定义极为简洁——一个 Markdown 文件即可：

```yaml
# ~/.claude/agents/backend-architect.md
name: backend-architect
description: 设计 RESTful API、微服务边界和数据库 schema
model: sonnet
tools: [Read, Write, Edit, Bash]

You are a senior backend architect...
```

**作用域**：
- `.claude/agents/` — 项目级 Agent
- `~/.claude/agents/` — 用户级 Agent（全局）

**加载机制**：启动时自动扫描目录下的 `.md` 文件，无需显式注册。

---

## 专题子模块

| 子模块 | 内容 |
|--------|------|
| [基础架构](architecture/) | Orchestrator-Worker 分层隔离架构、上下文隔离、权限模型 |
| [MCP 协议](mcp/) | Model Context Protocol 详解、生态、安全模型 |
| [Ultra Plan](ultra-plan/) | 三探索 + 一评审的并行探索机制 |

---

## 状态与记忆管理

| 记忆层级 | 实现方式 | 生命周期 |
|---------|---------|---------|
| **短期记忆** | 各 Agent 独立的上下文窗口 | 单次任务 |
| **中期记忆** | `CLAUDE.md` 文件 | 项目级，跨会话 |
| **长期记忆** | 外部 MCP 向量数据库 | 持久化 |

### CLAUDE.md：项目级共享上下文

```markdown
# 项目上下文

## 技术栈
- 前端：React + TypeScript + Vite
- 后端：FastAPI + PostgreSQL

## 代码规范
- 使用函数式组件，避免 class 组件
- API 调用统一放在 `services/` 目录
```

---

## AGENTS.md：跨工具标准

正在成为工具无关的 Agent 配置标准：

```markdown
# AGENTS.md

## 团队成员

### backend-architect
- 职责：API 设计、数据库 schema
- 不处理：前端代码、UI 设计

## 协作规则
1. 任何 API 变更必须经过 backend-architect 审查
2. 代码提交前必须通过测试
```

---

## 参考

- [Claude Code 官方文档](https://code.claude.com/docs)
- [Claude Code 基础架构](architecture/)
- [Claude Code MCP 协议](mcp/)
- [Claude Code Ultra Plan](ultra-plan/)
- [Claude Code 与 OpenClaw 架构对比](/research/claude-code-vs-openclaw/)
