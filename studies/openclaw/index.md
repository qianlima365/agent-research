---
title: OpenClaw 专题
permalink: /studies/openclaw/
description: 开源多通道 AI Agent 框架，通过统一 Gateway 接入 20+ 通信平台，让 Agent 成为真正的数字员工。
tags: [openclaw, multi-agent, cli, open-source]
---

# OpenClaw 专题

**官网/仓库**: 开源社区驱动 | **协议**: MIT License

> 从通信层向内构建的 Agent 平台——不是让 Agent 为你写代码，而是让 Agent 替你值守、通知、协作。

---

## 产品定位

OpenClaw 是一个**开源多通道 AI Agent 框架**，核心设计理念是**"广泛连接"**——通过统一的 WebSocket Gateway 将 AI Agent 接入 20+ 通信平台（WhatsApp、Telegram、Discord、Slack、飞书等），让 Agent 成为真正的"数字员工"。

与 Claude Code 的"垂直深度"不同，OpenClaw 追求"水平广度"：一个 Agent 可以同时出现在多个平台上，响应不同渠道的消息，与人类团队成员并肩工作。

---

## 三文件人格体系

OpenClaw 的 Agent 定义采用独特的**三文件分离**设计：

```
~/.openclaw/agency-agents/
├── planner/
│   ├── SOUL.md      ← 灵魂/核心人格
│   ├── AGENTS.md    ← 协作关系定义
│   └── IDENTITY.md  ← 身份信息与能力边界
```

| 文件 | 作用 | 被谁读取 |
|------|------|---------|
| **SOUL.md** | 核心人格、价值观、决策风格 | LLM（注入 system prompt） |
| **AGENTS.md** | 与其他 Agent 的协作契约 | 系统（权限控制） |
| **IDENTITY.md** | 能力清单、不处理事项 | LLM + 系统 |

**设计洞察**：AGENTS.md 可被系统解析并强制执行——如果 Agent A 未在 AGENTS.md 中声明对 Agent B 的依赖，系统会拒绝 `agentToAgent` 调用。这是**机器可读的权限控制**。

---

## 专题子模块

| 子模块 | 内容 |
|--------|------|
| [基础架构](architecture/) | 注册中心、消息总线、多平台 Gateway 的三层架构 |
| [记忆层设计](memory/) | VectorStore 长期记忆、Convex DB 状态持久化、Cron 常驻运行 |
| [多智能体设计](multi-agent/) | agentToAgent 协作原语、网状通信、去中心化模型 |

---

## 模型无关设计

支持 10 种 LLM 提供商，其中 7 种免 API key：

| 提供商 | 费用 |
|--------|------|
| Gemini CLI | 免费（1000 次/天） |
| OpenClaw CLI | 免费 |
| Hermes CLI | 免费 |
| Ollama | 免费（本地） |
| Claude Code | 需会员 |
| Copilot CLI | 需订阅 |
| Codex CLI | 需 Plus/Pro |

**策略**：通过 CLI 工具调用而非直接调用 API，复用用户已有的 AI 订阅，实现"零额外成本"。

---

## 开源生态

| 项目 | 定位 |
|------|------|
| **openclaw-agents** | 9 个预设 Agent 一键部署 |
| **Fleet** | 多 Agent 舰队管理 CLI |
| **Clawe** | 带 Web Dashboard 的多 Agent 协调系统 |
| **Clawion** | 基于文件的任务协调器 |

---

## 参考

- [Claude Code 与 OpenClaw 架构对比](/research/claude-code-vs-openclaw/)
- [OpenClaw 基础架构](architecture/)
- [OpenClaw 记忆层设计](memory/)
- [OpenClaw 多智能体设计](multi-agent/)
