---
title: Claude Code 专题研究
date: 2026-05-07
description: 深入解析 Anthropic Claude Code 的多智能体架构设计、MCP 工具协议、Orchestrator-Worker 协作模型和 Ultra Plan 并行探索机制。
tags: [claude-code, anthropic, mcp, multi-agent, ide-integration]
---

# Claude Code 专题研究

**开发者**: Anthropic（官方） | **协议**: 闭源（免费使用）

> 从 IDE 内向外的深度集成——不是又一个聊天窗口，而是理解你代码库的工程伙伴。

---

## 产品定位

Claude Code 是 Anthropic 推出的官方 CLI 工具，核心设计理念是**"深度集成"**——它不是独立运行的聊天机器人，而是嵌入在开发者工作流中的**代码感知型 AI 助手**。它能读取你的代码库、理解项目结构、执行命令、操作文件，并且支持多 Agent 协作。

与 OpenClaw 的"水平广度"不同，Claude Code 追求"垂直深度"：把代码开发这个场景做到极致，从代码生成到审查到重构，形成闭环。

---

## 核心架构：分层隔离

```
┌─────────────────────────────────────────────┐
│  Layer 4: Orchestrator（主会话调度器）         │
│  - 解析用户意图，决定调用哪个子 Agent           │
│  - 管理子 Agent 的生命周期（创建/销毁/暂停）    │
│  - 所有子 Agent 的输出汇总后返回给用户          │
├─────────────────────────────────────────────┤
│  Layer 3: SubAgent Pool（子 Agent 池）         │
│  - 每个子 Agent 有独立的 Context Window         │
│  - 权限分级：只读(default) / 读写 / 危险操作    │
│  - 子 Agent 之间不可直接通信                    │
├─────────────────────────────────────────────┤
│  Layer 2: MCP Client（工具中间层）              │
│  - 统一协议与外部工具交互                       │
│  - 工具调用前的权限检查                         │
│  - 结果序列化后注入子 Agent 上下文              │
├─────────────────────────────────────────────┤
│  Layer 1: Tool Ecosystem（工具生态）            │
│  - FileSystem / Git / Browser / Database ...   │
│  - 自定义 MCP Server                           │
└─────────────────────────────────────────────┘
```

### 四层分工

| 层级 | 职责 | 关键技术 |
|------|------|---------|
| **Orchestrator** | 意图理解与任务调度 | LLM 路由、上下文过滤 |
| **SubAgent Pool** | 专家执行 | 隔离上下文、权限受限 |
| **MCP Client** | 工具代理 | Model Context Protocol |
| **Tool Ecosystem** | 能力扩展 | 文件系统、Git、浏览器、数据库等 |

---

## Agent 定义：文件 + 目录约定

Claude Code 的 Agent 定义极为简洁——一个 Markdown 文件即可：

```yaml
# ~/.claude/agents/backend-architect.md
name: backend-architect
description: 设计 RESTful API、微服务边界和数据库 schema
model: sonnet
tools: [Read, Write, Edit, Bash]

You are a senior backend architect specializing in scalable system design...
```

**作用域**：
- `.claude/agents/` — 项目级 Agent，只在当前项目可用
- `~/.claude/agents/` — 用户级 Agent，全局可用

**加载机制**：启动时自动扫描目录下的 `.md` 文件，无需显式注册。

**与 OpenClaw 的对比**：Claude Code 的 Agent 定义是**扁平的**（一个文件搞定），OpenClaw 是**结构化的**（三文件分离）。扁平意味着上手快，结构化意味着可维护性强。

---

## Orchestrator-Worker 协作模型

### 通信机制：星型拓扑

```
    ┌─────────────────────────┐
    │    主 Claude 会话        │  ← Orchestrator
    └──────┬────────┬─────────┘
           │        │
     ┌─────┘        └─────┐
     ↓                    ↓
┌─────────┐         ┌─────────┐
│ Agent A │         │ Agent B │  ← Worker（隔离上下文）
│ 只读     │         │ 读写    │
└─────────┘         └─────────┘
```

**关键约束**：
- Agent 之间**不能直接对话**，所有通信必须通过 Orchestrator 中转
- 每个 Worker 只接收**任务相关的上下文片段**，而非完整会话历史
- 默认权限为**只读**，需 Orchestrator 显式授权才能写入

**设计理由**：安全性。在代码开发场景中，一个失控的 Agent 可能误删文件、引入安全漏洞。Orchestrator 作为"内核"，严格控制每个子进程的权限。

---

## MCP（Model Context Protocol）

MCP 是 Claude Code 最核心的架构创新，也是 Anthropic 推动的行业标准。

### 协议模型

```
┌─────────────┐         ┌─────────────┐
│  MCP Client │ ←────→ │  MCP Server │
│ (Claude Code)│  JSON-RPC │  (外部工具)  │
└─────────────┘         └─────────────┘
```

**通信方式**：JSON-RPC 2.0，标准化了工具发现、调用和结果返回的格式。

**生态现状**：社区已开发大量 MCP Server：
- 数据库：PostgreSQL、SQLite、MySQL
- 协作：Slack、Notion、GitHub
- 搜索：Brave Search、Exa
- 自动化：Zapier、Make

### 安全模型

| 操作类型 | 默认权限 | 需要确认 |
|---------|---------|---------|
| 读取文件 | ✅ 允许 | 否 |
| 写入文件 | ⚠️ 需授权 | 是 |
| 执行命令 | ⚠️ 需授权 | 是 |
| 删除文件 | ❌ 禁止 | 必须人类确认 |
| 网络请求 | ⚠️ 需授权 | 是 |

---

## Ultra Plan：三探索 + 一评审

Claude Code Ultra 订阅提供的多 Agent 模式：

```python
class UltraPlan:
    def solve(self, problem: str) -> str:
        # 1. 启动 3 个 Explorer Agent（并行）
        explorers = [
            SubAgent(strategy="保守"),
            SubAgent(strategy="激进"),
            SubAgent(strategy="创新"),
        ]
        solutions = parallel_execute(explorers, problem)

        # 2. 启动 Critic Agent（独立评审）
        critic = SubAgent(read_only=True)
        evaluation = critic.evaluate(solutions)

        # 3. 返回最优方案 + 评审理由
        return best_solution + evaluation
```

**设计洞察**：模拟人类团队的"头脑风暴 + 独立评审"流程。3 个 Explorer 提供多样性，Critic 避免自我偏差（让模型评审自己的输出容易产生偏见）。

**适用场景**：
- 架构设计（多个方案对比）
- 代码重构（不同重构策略评估）
- 技术选型（多维度打分）

---

## 状态与记忆管理

| 记忆层级 | 实现方式 | 生命周期 |
|---------|---------|---------|
| **短期记忆** | 各 Agent 独立的上下文窗口 | 单次任务 |
| **中期记忆** | `CLAUDE.md` 文件 | 项目级，跨会话 |
| **长期记忆** | 外部 MCP 向量数据库 | 持久化 |

### CLAUDE.md：项目级共享上下文

在项目根目录放置 `CLAUDE.md`，Claude Code 会自动读取并注入到每次对话的上下文中：

```markdown
# 项目上下文

## 技术栈
- 前端：React + TypeScript + Vite
- 后端：FastAPI + PostgreSQL
- 部署：Docker + AWS ECS

## 代码规范
- 使用函数式组件，避免 class 组件
- API 调用统一放在 `services/` 目录
- 错误处理必须包含用户友好的中文提示

## 已知问题
- 用户认证模块正在重构，不要修改相关代码
```

**价值**：相当于项目的"入职手册"，新会话不需要重复交代背景信息。

---

## AGENTS.md：跨工具标准

AGENTS.md 正在成为工具无关的 Agent 配置标准（Claude Code、Codex CLI、OpenClaw 都在支持）：

```markdown
# AGENTS.md

## 团队成员

### backend-architect
- 职责：API 设计、数据库 schema、微服务边界
- 不处理：前端代码、UI 设计

### frontend-specialist
- 职责：组件设计、状态管理、性能优化
- 不处理：后端逻辑、数据库操作

## 协作规则
1. 任何 API 变更必须经过 backend-architect 审查
2. UI 组件必须通过 accessibility 检查
3. 代码提交前必须通过测试
```

---

## 优势与局限

### 优势

- **代码感知最强**：深度理解代码库结构、依赖关系、类型定义
- **MCP 生态最完善**：标准化的工具集成协议，社区活跃
- **安全性最高**：只读默认、权限分级、危险操作人类确认
- **IDE 集成最深**：不仅是 CLI，还可嵌入 VS Code、Cursor 等编辑器
- **Ultra Plan 模式**：3 Explorer + 1 Critic 的并行探索机制

### 局限

- **模型锁定**：深度绑定 Claude 系列，无法切换其他模型
- **场景单一**：专注代码开发，非技术场景支持弱
- **成本**：Ultra Plan 需要订阅，高频使用费用不低
- **Agent 间通信受限**：必须通过 Orchestrator 中转，灵活性不足
- **非常驻**：Agent 按需启动，不适合值守场景

---

## 适用场景

| 场景 | 匹配度 | 原因 |
|------|--------|------|
| 代码审查与重构 | ★★★★★ | 代码感知 + MCP 工具链是核心竞争力 |
| 复杂任务分解 | ★★★★★ | Orchestrator 模式天然适合拆分 |
| 快速原型开发 | ★★★★★ | 代码生成、测试、调试闭环 |
| 架构设计 | ★★★★★ | Ultra Plan 模式提供多方案对比 |
| 7×24 值守 | ★★☆☆☆ | 非常驻运行，无 Cron 调度 |
| 跨团队协作 | ★★★☆☆ | 无原生 Slack/Discord 集成，需 MCP 桥接 |
| 多模型实验 | ★☆☆☆☆ | 仅支持 Claude 系列 |

---

## 参考

- [Claude Code 官方文档](https://code.claude.com/docs)
- [Claude Code Ultra Plan 多 Agent 架构](https://www.mindstudio.ai/blog/claude-code-ultra-plan-multi-agent-architecture/)
- [Model Context Protocol 规范](https://modelcontextprotocol.io/)
- [AGENTS.md 工具无关标准](https://github.com/affaan-m/everything-claude-code/blob/main/.codex/AGENTS.md)
- [Claude Code 与 OpenClaw 架构对比](/research/claude-code-vs-openclaw/)
