---
title: The Agency / agency-agents
date: 2026-05-07
---

# The Agency / agency-agents

**仓库**: [msitarzewski/agency-agents](https://github.com/msitarzewski/agency-agents)
[中文版](https://github.com/jnMetaCode/agency-agents-zh)

> "A complete AI agency at your fingertips"

---

## 概述

agency-agents 是一个收录了 **144+ 个专业 AI Agent 人格** 的开源项目，旨在为各种 AI 编程助手和工具提供深度专业化的角色提示。与市面上泛滥的"假装你是一个开发者"这类通用提示不同，该项目提供的是**经过精心打磨、具备鲜明个性和完整工作流程的专家级 Agent**。

项目起源于一个 Reddit 讨论帖，迅速发展为覆盖 12 大职能领域、支持 11 种主流 AI 工具的 Agent 生态系统。

---

## 核心设计理念

每个 Agent 都包含五个核心要素：

| 要素 | 说明 |
|------|------|
| **鲜明个性** | 不是冰冷的指令，而是有态度、有风格的专家人格 |
| **明确交付物** | 每个 Agent 清楚知道自己要产出什么 |
| **成功指标** | 定义了衡量工作质量的具体标准 |
| **成熟工作流** | 内置经过验证的处理流程和方法论 |
| **学习记忆** | 能在交互中不断积累和进化 |

---

## 组织架构：12 大部门

项目按职能将 Agent 划分为 12 个部门，总计 144+ 个角色：

### 1. Engineering（工程）
25+ 个角色，覆盖前端、后端、DevOps、安全、数据工程等全栈领域。

### 2. Design（设计）
UX/UI、品牌设计、视觉设计等创意角色。

### 3. Paid Media（付费媒体）
PPC 竞价、程序化广告采买、追踪与归因专家。

### 4. Sales（销售）
Outbound 策略、Discovery 教练、赢单策略师等。

### 5. Marketing（营销）
含大量中国市场专精角色——小红书运营、微信公众号管理、抖音策略师、知乎策略师、百度 SEO 等。

### 6. Product（产品）
产品经理、用户研究、需求优先级排序、路线图规划。

### 7. Project Management（项目管理）
项目牧羊人、工作室制片人、流程优化师。

### 8. Testing（测试）
QA、性能基准、无障碍审核、测试结果分析。

### 9. Support（支持）
数据分析、财务追踪、基础设施运维。

### 10. Spatial Computing（空间计算）
XR 沉浸式开发、visionOS 空间工程师、Unity/Unreal 多人游戏架构等前沿角色。

### 11. Specialized（专业领域）
法律、医疗、供应链、政务数字化等垂直领域专家。

### 12. Academic（学术）
人类学家、地理学家、历史学家、心理学家、叙事学家——专为游戏/故事世界构建中创造**文化上连贯自洽的社会**而设计。

### 13. Game Development（游戏开发）
唯一按引擎细分目录的部门——Unity、Unreal Engine、Godot、Blender、Roblox Studio，每个引擎下都有着色器开发者、技术美术、多人游戏工程师等细分角色。

### 14. Finance（金融）
簿记、FP&A、投资研究、税务策略。

---

## 多工具集成：11 种主流平台

| 工具 | 格式 | 安装路径 |
|------|------|---------|
| **Claude Code** | 原生 `.md` | `~/.claude/agents/` |
| **GitHub Copilot** | 原生 `.md` | `~/.github/agents/` |
| **Gemini CLI** | `SKILL.md` | `~/.gemini/extensions/` |
| **Cursor** | `.mdc` rules | `.cursor/rules/` |
| **Aider** | `CONVENTIONS.md` | `./CONVENTIONS.md` |
| **Windsurf** | `.windsurfrules` | `./.windsurfrules` |
| **OpenCode** | `.md` | `.opencode/agents/` |
| **OpenClaw** | `SOUL.md` + `AGENTS.md` | `~/.openclaw/` |
| **Qwen Code** | `.md` SubAgents | `~/.qwen/agents/` |
| **Kimi Code** | YAML agent specs | `~/.config/kimi/agents/` |
| **Antigravity** | `SKILL.md` | `~/.gemini/antigravity/skills/` |

项目提供 `./scripts/install.sh` 脚本，支持交互式安装（自动检测已安装工具）、指定工具安装、CI 非交互模式，以及并行处理加速。

---

## 独特亮点

### 中国市场深度覆盖
在众多 AI Agent 项目中，agency-agents 对中国本土平台的覆盖尤为突出——从抖音算法、小红书种草、微信私域到快手老铁文化，都有专门的角色设计。

### 学术部门用于世界构建
人类学家、心理学家、地理学家、历史学家等学术 Agent 不是为了写论文，而是为了帮助创作者构建**有生活气息、文化上连贯自洽**的虚构世界——这对游戏叙事设计师和小说作者极具价值。

### 游戏开发引擎细分
按引擎（Unity/Unreal/Godot/Roblox/Blender）细分目录的做法非常罕见，每个引擎下还进一步按专业方向拆分（着色器、多人网络、关卡设计、音频工程等），体现了对游戏工业管线的深度理解。

### 多 Agent 协作场景
`examples/` 目录包含多个真实的多 Agent 协作场景文档，例如：
- 初创公司 MVP 开发（8 个 Agent 并行工作）
- 营销活动全案执行
- 企业级功能交付
- 付费媒体账户接管
- 完整的产品发现流程

---

## 技术信息

| 属性 | 详情 |
|------|------|
| 语言 | Shell (96.8%) + PowerShell (3.2%) |
| 许可 | MIT |
| 依赖 | 零依赖 — 纯提示词/Agent 文件分发 |
| 规模 | 10,000+ 行，144+ Agent |

---

## 适用场景

- **个人开发者**：为日常编码工作配备专业顾问（前端、安全、性能优化等）
- **小团队**：用多 Agent 协作完成 MVP 或营销活动
- **内容创作者**：利用中国市场专家 Agent 制定出海策略
- **游戏工作室**：用学术 Agent 构建世界观，用引擎专家 Agent 提升技术管线
- **企业**：标准化团队工作流程，将最佳实践编码为 Agent 人格