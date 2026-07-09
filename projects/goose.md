---
title: Goose
date: 2026-07-09
description: Linux Foundation AAIF 旗下开源、本地优先的通用 AI Agent，支持 Desktop / CLI / API 三种形态，基于 Rust 构建，兼容 15+ 模型提供商与 70+ MCP 扩展。
tags: [ai-agent, open-source, mcp, rust, aaif]
---

# Goose

**仓库**: [github.com/aaif-goose/goose](https://github.com/aaif-goose/goose)  
**官方文档**: [goose-docs.ai](https://goose-docs.ai/)  
**原属组织**: Block（Square）→ 2026 年 4 月捐赠给 **Linux Foundation Agentic AI Foundation（AAIF）**

> “Your native open source AI agent — desktop app, CLI, and API — for code, workflows, and everything in between.”

---

## 概述

Goose 是一个**开源、本地优先、可扩展的通用 AI Agent**，最初由 Block（前 Square）开发，2026 年 4 月随项目迁移至 Linux Foundation 的 Agentic AI Foundation（AAIF），成为与 Anthropic 的 MCP、OpenAI 的 AGENTS.md 并列的 AAIF 旗舰项目之一。

它强调“不仅仅是代码助手”：用户可以用它完成研究、写作、自动化、数据分析等多种任务，并通过 Desktop 应用、CLI 或 API 三种形态与 Agent 交互。

| 属性 | 详情 |
|------|------|
| 许可证 | Apache-2.0 |
| 主要语言 | Rust（约 66.2%）|
| Stars | ~50.9k |
| Forks | ~5.5k |
| Commits | 5,011+ |
| Releases | 141 |
| 最新版本 | v1.41.0（2026-07-03）|

---

## 核心特性

1. **三种使用形态**
   - **Desktop App**：原生桌面应用，支持 macOS、Linux、Windows。
   - **CLI**：完整终端工作流。
   - **API**：可嵌入其他应用或服务。

2. **多模型支持**
   - 支持 Anthropic、OpenAI、Google、Ollama、OpenRouter、Azure、Bedrock 等 **15+ 家模型提供商**。
   - 可通过 API key 接入，也可复用现有的 Claude / ChatGPT / Gemini 订阅（通过 ACP）。

3. **MCP 扩展生态**
   - 通过 **Model Context Protocol（MCP）** 连接 **70+ 扩展**，把文件系统、数据库、浏览器、GitHub 等外部工具变成 Agent 可调用的能力。

4. **本地与离线能力**
   - 支持 Ollama 等本地模型，适合隐私敏感或离线场景。

5. **通用任务**
   - 官方定位为通用 Agent，可用于代码、研究、写作、自动化、数据分析等多种任务。

---

## 架构与技术栈

- **核心语言**: Rust，强调性能与可移植性。
- **桌面端**: Electron GUI（位于 `ui/desktop`）。
- **协议层**:
  - **MCP（Model Context Protocol）**: 用于扩展/工具接入。
  - **ACP（Agent Client Protocol）**: 用于模型提供商接入，允许复用现有订阅。
- **会话模型**: CLI 支持 `goose session`，并提供 `/status`、`/model`、`/goal` 等 slash 命令。
- **配置**: CLI 与 Desktop 共享同一套设置（provider、model、extensions）。

> 公开文档中关于运行时沙箱、进程隔离、安全模型的细节较少，整体采用“本地运行、用户授权”模式。

---

## 最新进展（v1.41.0，2026-07-03）

- **新增模型提供商**: iFlytek Spark、Astron MaaS、Fireworks AI、Perplexity、阿里云 DashScope/Qwen、Databricks AI Gateway、Scaleway、NEAR AI Cloud、xAI SuperGrok OAuth 等。
- **CLI 增强**: 新增 `/status`、`/model`、`/goal` slash 命令；`--edit` 会话标志；图片读取工具；会话导入；`goose://resume` 深度链接。
- **桌面端**: 扩展法语、德语、意大利语、葡萄牙语、印尼语、马来语、越南语、繁体中文等本地化支持。
- **架构调整**: provider 代码重构为 `goose-providers` crate；UI 直接连接 ACP 而非 `goosed`；MCP 通过 tool streams 路由。
- **模型注册**: 新增 `claude-sonnet-5` 等模型。

---

## 治理与社区

Goose 采用**轻量级技术治理模型**，由 Linux Foundation 项目框架托管：

- **核心价值观**: Open（公开透明）、Flexible（支持远程前沿与本地私有模型）、Choice（避免锁定单一模型/协议/栈）。
- **角色**: Contributors、Maintainers、Core Maintainers 三级。
- **决策流程**: 日常变更通过 PR/Discord 共识；重大变更需公开提案、至少一周社区审议、多数 Core Maintainer 批准；项目创始人 Bradley Axen 为最终僵局仲裁者。
- **贡献要求**: 鼓励从小 PR 开始建立信任；使用 Hermit + just；Rust 代码需通过 `cargo check/test/fmt/clippy`；PR 标题遵循 Conventional Commits；使用 Codex 进行 AI review。

---

## 与同类工具的对比

| 维度 | Goose | Claude Code | Cline | Aider | OpenCode |
|------|-------|-------------|-------|-------|----------|
| 开源 | ✅ Apache-2.0 | ❌ 闭源 | ✅ | ✅ | ✅ |
| 模型支持 | 15+ 提供商 + 本地 | 仅 Claude | 多模型 | 多模型 | 75+ 提供商 |
| 交互形态 | Desktop / CLI / API | CLI | VS Code 插件 | CLI | Desktop / CLI |
| MCP 支持 | ✅ 扩展式 | ✅ 原生 | 部分 | 插件 | ✅ |
| 本地/离线 | ✅ Ollama | ❌ | ✅ | ✅ | ✅ |
| 最佳场景 | 模型灵活、本地部署、开源扩展 | 高可靠性、Anthropic 生态 | VS Code 前端/全栈 | 大仓库重构、Git-first | 成本与性能平衡 |

第三方 Agentic CLI 基准参考（AIMultiple）显示：

| 工具 | 综合得分 | 单次任务成本 |
|------|----------|--------------|
| OpenCode | 81.6% | $1.03 |
| Claude Code | 较高 | $1.83 |
| Goose | 62.5% | $3.23 |
| Cline | 64.4% | 浮动 |

> 该基准仅供参考，测试场景有限。Goose 的优势更体现在开源可控、多模型与本地部署，而非单纯的 benchmark 成本/得分。

---

## 适用场景

**适合 Goose 的场景：**
- 需要在本地或隔离环境运行的 Agent（隐私/合规）。
- 想避免绑定单一模型厂商，灵活切换 Claude / GPT / Gemini / 本地模型。
- 需要把 Agent 嵌入自有工作流（提供 API）。
- 重视开源可审计、可二次开发。

**可能不适合的场景：**
- 追求开箱即用、极高自动执行可靠性（Claude Code 或 OpenCode 可能更稳）。
- 纯 VS Code 重度用户（Cline 更自然）。
- 超大规模代码库重构（Aider 的 repository map 更有优势）。

---

## 参考来源

- [GitHub - aaif-goose/goose](https://github.com/aaif-goose/goose)
- [Goose 官方文档](https://goose-docs.ai/)
- [Goose v1.41.0 Release Notes](https://github.com/aaif-goose/goose/releases/tag/v1.41.0)
- [GOVERNANCE.md](https://github.com/aaif-goose/goose/blob/main/GOVERNANCE.md)
- [CONTRIBUTING.md](https://github.com/aaif-goose/goose/blob/main/CONTRIBUTING.md)
- [OSS Insight - aaif-goose/goose](https://ossinsight.io/analyze/aaif-goose/goose)
- [AAIF 成立新闻稿](https://aaif.io/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation-aaif-anchored-by-new-project-contributions-including-model-context-protocol-mcp-goose-and-agentsmd/)
- [Goose 迁移至 AAIF 公告](https://goose-docs.ai/blog/2026/04/07/goose-moves-to-aaif/)
- [Phos AI Labs: Claude Code vs Goose](https://phosailabs.com/blog/claude-code-vs-goose)
- [LowCode Agency: Claude Code vs Goose](https://www.lowcode.agency/blog/claude-code-vs-goose)
- [AIMultiple: Agentic CLI Tools Compared](https://research.aimultiple.com/agentic-cli/)
- [Agentlist: Claude Code vs Cline vs Aider](https://www.agentlist.top/en/articles/claude-code-cline-aider-coding-agent-comparison/)
