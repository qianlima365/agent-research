---
title: Agency Orchestrator — AI 工作流编排引擎
date: 2026-05-07
description: 基于 DAG 的 AI 工作流编排引擎，一句话自动调度多个专家角色协作. 对 agency-agents 角色库的封装, 解决怎么做的问题。
tags: [multi-agent, orchestration, dag, yaml]
---

# Agency Orchestrator — AI 工作流编排引擎

**仓库**: [jnMetaCode/agency-orchestrator](https://github.com/jnMetaCode/agency-orchestrator)

> "一句话调度多个 AI 专家自动协作，几分钟交付完整方案"

---

## 概述

Agency Orchestrator 是一个基于 **YAML 配置 + DAG 自动并行执行** 的多智能体编排工具。它解决了多 Agent 协作中最实际的痛点：**让多个专业 Agent 角色按照预定流程高效配合，而不需要写代码**。

与 agency-agents（角色库）形成互补关系：
- **agency-agents** = "谁来做"（211 个专业角色定义）
- **agency-orchestrator** = "怎么做"（DAG 工作流编排引擎）

---

## 核心定位

| 维度 | 传统方式 | Agency Orchestrator |
|------|---------|-------------------|
| 角色定义 | 自己写 Prompt | 复用 211 个现成角色 |
| 流程编排 | 写 Python（LangGraph 等） | 写 YAML 或直接一句话 |
| API 费用 | 必须付费 | **7 种方式免 API key** |
| 依赖安装 | pip + 几十个包 | **npm + 2 个依赖** |
| 并行执行 | 手动建图 | **DAG 自动检测并行步骤** |

---

## 架构设计

### DAG 执行引擎

```
输入需求 → 解析 YAML → 构建 DAG → 自动检测可并行节点 → 并发执行 → 汇总输出
```

示例工作流（自动识别并行）：

```yaml
steps:
  - id: analyze
    role: engineering/product-analyst
    prompt: "分析需求"

  - id: tech_review
    role: engineering/software-architect
    prompt: "技术评审"
    depends_on: [analyze]     # 依赖 analyze 的输出

  - id: design_review
    role: design/ux-reviewer
    prompt: "设计评审"
    depends_on: [analyze]     # 也依赖 analyze，与 tech_review 并行

  - id: summary
    role: product/pm
    prompt: "汇总结论"
    depends_on: [tech_review, design_review]  # 等两者都完成
```

引擎自动构建 DAG：`analyze` → `tech_review` + `design_review`（并行）→ `summary`

### 关键执行特性

| 特性 | 说明 |
|------|------|
| **变量传递** | `{{analyze.output}}` 在步骤间传递结果 |
| **并发控制** | `concurrency` 参数限制最大并行数 |
| **失败重试** | 指数退避，超时自动递增 |
| **断点续跑** | `--resume last` 从上次中断处继续 |
| **循环迭代** | `loop` 配置支持 back_to、max_iterations、exit_condition |
| **条件分支** | `condition` 表达式控制步骤是否执行 |
| **人工审批** | `type: "approval"` 节点暂停等待人类确认 |

---

## 一句话编排：`ao compose`

最核心的创新是 `ao compose` 命令：

```bash
# 一句话描述需求，AI 自动完成：选角色 → 设计 DAG → 生成 YAML → 执行
ao compose "帮我分析做一个 AI 记账工具的可行性" --run

# 技术选型报告
ao compose "对比 Cursor、Windsurf 和 Copilot" --run

# 商业计划书
ao compose "用 10 万块启动一个 AI 教育项目" --run
```

`--run` 标志让编排和执行一步到位。不加 `--run` 则只生成 YAML 供人工审核。

---

## LLM 支持：10 种提供商，7 种免 API key

| 提供商 | 类型 | 费用 |
|--------|------|------|
| claude-code | CLI | 需 Claude 会员，无额外费用 |
| gemini-cli | CLI | 免费（1000 次/天） |
| copilot-cli | CLI | 需 Copilot 订阅 |
| codex-cli | CLI | 需 ChatGPT Plus/Pro |
| openclaw-cli | CLI | 免费 |
| hermes-cli | CLI | 免费（开源） |
| ollama | 本地 | 免费（本地模型） |
| deepseek | API | 需 API key |
| claude | API | 需 ANTHROPIC_API_KEY |
| openai | API | 需 OPENAI_API_KEY |

**关键洞察**：通过 CLI 工具调用（而非直接调用 API），可以充分利用用户已有的 AI 订阅，实现"零额外成本"的多智能体协作。

---

## 与 agency-agents 的协同

Agency Orchestrator 直接复用 agency-agents（及其中文版 agency-agents-zh）的角色库：

```
agency-agents/
├── engineering/
│   ├── engineering-software-architect.md
│   ├── engineering-frontend-specialist.md
│   └── ...
├── design/
├── product/
├── marketing/
│   ├── marketing-xiaohongshu-expert.md     # 小红书专家
│   ├── marketing-douyin-strategist.md      # 抖音策略师
│   └── ...
└── ...
```

角色路径直接映射到 YAML 中的 `role` 字段：`role: marketing/xiaohongshu-expert`

---

## 内置工作流模板（32 个）

| 类别 | 代表模板 |
|------|---------|
| 开发类 | PR 评审、技术债务审计、安全审计、发布检查清单 |
| 营销类 | 竞品分析、小红书种草笔记、SEO 内容矩阵 |
| 数据/运维 | 数据管道评审、事故复盘、周报生成 |
| 战略/法务 | 商业计划书、合同审查 |
| 通用类 | 产品需求评审、协作小说创作、CEO 组织架构协作 |

---

## MCP Server 模式

Agency Orchestrator 可以启动为 MCP Server，被其他工具调用：

```bash
ao serve
```

暴露的 MCP 工具：
- `run_workflow` — 执行工作流
- `validate_workflow` — 校验 YAML
- `list_workflows` — 列出可用模板
- `explain_workflow` — 用自然语言解释计划

这意味着 Claude Code、Cursor、Copilot 等工具可以通过 MCP 直接调用 Agency Orchestrator 的能力。

---

## 输出结构

```
ao-output/可行性分析-20260507/
├── summary.md          # 最终汇总输出
├── steps/
│   ├── 1-analyze.md    # 每个步骤的完整输出
│   ├── 2-tech_review.md
│   └── 3-summary.md
└── metadata.json       # 耗时、token 用量、步骤状态
```

---

## 技术信息

| 属性 | 详情 |
|------|------|
| 核心语言 | TypeScript (72.8%) |
| 脚本 | Shell (19.6%), JavaScript (7.2%) |
| 许可证 | Apache-2.0 |
| 依赖 | npm + 2 个包（极简设计） |
| 配置 | YAML 零代码 |

---

## 适用场景

- **创业者**：快速验证商业想法，自动生成可行性分析报告
- **产品经理**：PRD 评审、竞品分析、需求拆解
- **开发者**：技术选型、代码审查、架构评审
- **内容创作者**：深度长文写作、多平台内容矩阵规划
- **独立开发者**：一人公司全岗位协作（产品 + 设计 + 开发 + 营销）

---

## 生态关系

Agency Orchestrator 是 agency-agents 生态的"调度层"：

```
[agency-agents-zh]  ←── 提供 211 个中文角色
         ↓
[agency-orchestrator] ←── YAML 编排 + DAG 执行引擎
         ↓
[superpowers-zh]     ←── 20 个工作方法论 skills
         ↓
[ai-coding-guide]    ←── 实战教程
```

---

## 参考

- [Agency Orchestrator 仓库](https://github.com/jnMetaCode/agency-orchestrator)
- [agency-agents-zh 中文角色库](https://github.com/jnMetaCode/agency-agents-zh)
- [superpowers-zh 工作方法论](https://github.com/jnMetaCode/superpowers-zh)
- [shellward 安全中间件](https://github.com/jnMetaCode/shellward)
