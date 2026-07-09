---
title: Pi
date: 2026-07-09
description: Earendil Works 旗下开源、极简、可扩展的 TypeScript AI Agent Harness，核心只提供 4 个基础工具，其余能力通过 TypeScript 扩展构建。
tags: [ai-agent, open-source, typescript, agent-harness, extensible]
---

# Pi / pi-mono

**仓库**: [github.com/earendil-works/pi](https://github.com/earendil-works/pi)  
**官方网站**: [pi.dev](https://pi.dev/)  
**原属仓库**: `badlogic/pi-mono`（Mario Zechner，libGDX 作者）  
**现属组织**: Earendil Works

> “Minimal agent harness — adapt Pi to your workflows, not the other way around.”

---

## 概述

Pi 是一个**极简、可扩展的 TypeScript AI Agent Harness**。它最初由 Mario Zechner 在 `badlogic/pi-mono` 下开发，2026 年 5 月迁移至 `earendil-works/pi`，npm 包名也从 `@mariozechner/*` 切换为 `@earendil-works/*`。

与大多数“开箱即用”的编码 Agent 不同，Pi 采用 **“反框架”** 设计：核心只提供四个基础工具（`read`、`write`、`edit`、`bash`），其他能力都通过 TypeScript 扩展按需构建。

| 属性 | 详情 |
|------|------|
| 许可证 | MIT |
| 主要语言 | TypeScript（约 93.7%）|
| Stars | ~69k |
| Forks | ~8.5k |
| Commits | 4,874+ |
| 最新版本 | v0.80.3（2026-06-30）|
| npm 包名 | `@earendil-works/pi-coding-agent` |

---

## 核心特性

1. **极简核心，四类基础工具**
   - `read`、`write`、`edit`、`bash`，其余全部交给扩展。

2. **四种运行模式**
   - **交互式 TUI**
   - **打印/JSON 模式** (`pi -p "query"`)
   - **RPC 模式**（stdin/stdout）
   - **SDK 嵌入**

3. **统一多模型 API**
   - `pi-ai` 包支持 OpenAI、Anthropic、Google 等 **15–20+ 家提供商**。
   - 支持 **会话中切换模型**。

4. **树状会话历史**
   - 会话以 **append-only JSONL** 持久化，节点通过 `id/parentId` 形成树。
   - 支持分支 `/fork`、历史树 `/tree`、上下文压缩 `/compact`。

5. **自扩展能力**
   - 扩展是 TypeScript 模块，可注册工具、slash 命令、快捷键、UI 组件、事件钩子。
   - 支持热重载 `/reload-plugins`：Agent 可以修改自己的扩展代码并立即生效。

6. **上下文工程**
   - 通过 `AGENTS.md`、`SYSTEM.md`、skills、prompt templates、动态上下文注入来控制行为。

---

## 架构与包结构

Pi 是 npm monorepo，核心包职责清晰：

| 包 | 职责 |
|---|---|
| `@earendil-works/pi-ai` | 统一 LLM 提供商 API |
| `@earendil-works/pi-agent-core` | Agent 循环与消息状态管理 |
| `@earendil-works/pi-coding-agent` | 交互式 CLI、工具、会话、扩展系统 |
| `@earendil-works/pi-tui` | 终端 UI 组件与差分渲染 |

依赖关系：`coding-agent` → `agent` → `ai`；`tui` 相对独立。

会话模型：
- `AgentSession` 管理生命周期、持久化、上下文压缩。
- 历史是树结构，支持分支而不修改原历史。

扩展模型：
- 扩展可监听 `session_start`、`before_agent_start`、`tool_call` 等事件。
- 可注入上下文、拦截工具调用、注册自定义 CLI flag。

---

## 安全与隔离

Pi **没有内置权限系统**，默认以启动用户的权限运行。官方推荐通过外部隔离来保障安全：

- **Gondolin 扩展**：Agent 与 API key 留在宿主机，工具和 shell 命令路由到本地 micro-VM。
- **Docker**：将整个 `pi` 进程放入容器，但 API key 需进入容器。
- **OpenShell**：策略控制的沙箱，可限制文件系统、进程、网络、凭证和推理。

供应链安全：
- 外部依赖锁定到精确版本；工作区包使用范围版本。
- `.npmrc` 设置 `save-exact=true`、`min-release-age=2`。
- lockfile 是单一事实来源；变更 lockfile 必须设置 `PI_ALLOW_LOCKFILE_CHANGE=1`。
- 发布 CLI 附带 `npm-shrinkwrap.json` 锁定传递依赖。

---

## 最新进展（v0.80.3，2026-06-30）

- **新增模型支持**：Anthropic Claude Sonnet 5，支持自适应 thinking；Azure Foundry 风格端点。
- **CLI/TUI**：`outputPad` 输出间距、`externalEditor` Ctrl+G 编辑器控制、`/debug` 命令。
- **RPC**：新增 `get_entries` 与 `get_tree`，支持仅 RPC 启动 `./rpc-entry`。
- **扩展事件**：`session_info_changed` 事件。
- **默认模型**：OpenAI 默认模型改为 `gpt-5.5`。
- **修复**：Claude Sonnet 5 thinking payload、Provider HTTP 错误带响应体、BMP 转 PNG、TUI/压缩/资源通知崩溃等问题。

---

## 治理与贡献

- **价值观**：Open、Flexible、Choice（避免锁定单一模型/协议/栈）。
- **AGENTS.md**：详细的 AI Agent 协作规范，包括沟通风格、代码质量、命令、依赖安全、Git、Issue/PR、Changelog、发布流程等。
- **开发流程**：
  - 安装：`npm install --ignore-scripts`
  - 构建：`npm run build`
  - 检查：`npm run check`
  - 非 LLM 测试：`./test.sh`
  - 源码运行：`./pi-test.sh`
- **发布**：采用 **Lockstep versioning**，所有包同步版本号。

---

## 与同类工具的对比

| 维度 | Pi | Claude Code | Goose | Aider | Cline |
|------|----|-------------|-------|-------|-------|
| 开源 | ✅ MIT | ❌ 闭源 | ✅ Apache-2.0 | ✅ | ✅ |
| 核心定位 | Agent 构建套件 / 反框架 | 终端编码 Agent | 通用本地 Agent | Git-first 编码助手 | VS Code 插件 |
| 内置工具 | 仅 4 个，其余扩展 | 丰富 | 丰富 | 丰富 | 丰富 |
| 扩展性 | 极强（TypeScript 模块）| 有限 | MCP 扩展 | 插件/脚本 | 扩展市场 |
| 模型支持 | 15–20+，可会话中切换 | 仅 Claude | 15+ | 多模型 | 多模型 |
| 会话历史 | 树状分支 | 线性 | 线性 | 线性 | 线性 |
| IDE 集成 | 无 | 无 | 无 | 无 | ✅ VS Code |
| 本地/离线 | 可通过扩展支持 | ❌ | ✅ Ollama | ✅ | ✅ |

### 性能参考

Pi 在 TerminalBench 等第三方基准中表现靠前：尽管核心极简，但配合 Claude Opus 4.5 曾排名第二。

> 注意：Pi 的“反框架”设计意味着它把许多功能（MCP、子 Agent、计划模式、权限门控、后台 bash、待办清单）留作扩展练习，而不是内置。

---

## 适用场景

**适合 Pi 的场景：**
- 你想深度定制 Agent 行为，而不是接受厂商预设工作流。
- 你需要把 Agent 嵌入自己的工具链或 CI/CD（RPC/SDK 模式）。
- 你偏好 TypeScript/npm 生态，希望扩展就是普通 TS 模块。
- 你需要树状会话历史、会话中切换模型等高级交互。

**可能不适合的场景：**
- 追求开箱即用、功能齐全的终端编码 Agent（Claude Code / Goose 更直接）。
- 不想自己写扩展来实现 MCP、子 Agent、权限控制。
- 你主要在 VS Code 中工作（Cline 更自然）。

---

## 参考来源

- [GitHub - earendil-works/pi](https://github.com/earendil-works/pi)
- [Pi 官网](https://pi.dev/)
- [Pi Has a New Home at Earendil](https://pi.dev/news/2026/5/7/pi-has-a-new-home)
- [npm - @earendil-works/pi-coding-agent](https://www.npmjs.com/package/@earendil-works/pi-coding-agent)
- [Pi Architecture Docs](https://pt-act-pi-mono.mintlify.app/concepts/architecture)
- [Containerization Guide](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/containerization.md)
- [GitHub Releases v0.80.3](https://github.com/earendil-works/pi/releases/tag/v0.80.3)
- [AGENTS.md](https://github.com/earendil-works/pi/blob/main/AGENTS.md)
- [development.md](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/development.md)
- [Pi Mono Explained](https://hoangyell.com/pi-mono-explained/)
