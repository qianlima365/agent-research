---
title: OpenClaw 专题研究
date: 2026-05-07
description: 深入解析 OpenClaw 多通道 AI Agent 框架的架构设计、三文件人格体系、去中心化协作模型和开源生态。
tags: [openclaw, multi-agent, cli, open-source, gateway]
---

# OpenClaw 专题研究

**官网/仓库**: 开源社区驱动 | **协议**: MIT License

> 从通信层向内构建的 Agent 平台——不是让 Agent 为你写代码，而是让 Agent 替你值守、通知、协作。

---

## 产品定位

OpenClaw 是一个**开源多通道 AI Agent 框架**，核心设计理念是**"广泛连接"**——通过统一的 WebSocket Gateway 将 AI Agent 接入 20+ 通信平台（WhatsApp、Telegram、Discord、Slack、飞书等），让 Agent 成为真正的"数字员工"，而不是停留在 IDE 内的编程助手。

与 Claude Code 的"垂直深度"不同，OpenClaw 追求"水平广度"：一个 Agent 可以同时出现在多个平台上，响应不同渠道的消息，与人类团队成员并肩工作。

---

## 核心架构：注册中心 + 消息总线

```
┌─────────────────────────────────────────────┐
│  Layer 3: Gateway（多平台网关）                │
│  - WebSocket 统一接入层                        │
│  - 适配 WhatsApp/Discord/Slack/飞书 等 20+ 平台 │
│  - 消息格式标准化（入站/出站转换）              │
├─────────────────────────────────────────────┤
│  Layer 2: Agent Registry + Message Bus        │
│  - Agent 注册中心（agentId → Agent 实例）      │
│  - agentToAgent 工具（直接 RPC 调用）           │
│  - Cron Scheduler（定时唤醒调度器）             │
├─────────────────────────────────────────────┤
│  Layer 1: Agent Workers（Agent 工作池）        │
│  - 每个 Agent 加载 SOUL.md + AGENTS.md        │
│  - 独立进程/线程运行，常驻内存                  │
│  - 通过共享文件/Convex DB 持久化状态            │
└─────────────────────────────────────────────┘
```

### 三层分工

| 层级 | 职责 | 关键技术 |
|------|------|---------|
| **Gateway** | 平台适配 | WebSocket、各平台 SDK、消息标准化 |
| **Registry + Bus** | 服务发现与通信 | agentId 注册、agentToAgent RPC、Cron 调度 |
| **Agent Workers** | 业务执行 | LLM 调用、记忆检索、契约验证 |

---

## 三文件人格体系

OpenClaw 的 Agent 定义采用独特的**三文件分离**设计，将人格、协作、身份三个维度解耦：

```
~/.openclaw/agency-agents/
├── planner/
│   ├── SOUL.md      ← 灵魂/核心人格
│   ├── AGENTS.md    ← 协作关系定义
│   └── IDENTITY.md  ← 身份信息与能力边界
```

### SOUL.md — 灵魂层

定义 Agent 的核心人格、价值观和决策风格：

```markdown
# Planner 的灵魂

## 核心价值观
- 先规划后执行，不跳过分析直接给方案
- 面对不确定性时，倾向于收集更多信息而非猜测

## 决策风格
- 系统性：从目标 → 约束 → 方案 → 风险评估
- 保守倾向：默认选择风险最低的方案

## 语言特征
- 条理清晰，习惯用编号列表
- 关键结论加粗强调
```

### AGENTS.md — 协作层

定义与其他 Agent 的协作契约：

```markdown
# Planner 的协作契约

## 我能提供的输出
- 任务分解方案（含依赖关系图）
- 风险评估报告
- 资源需求估算

## 我需要从谁获取输入
- 需求澄清：产品经理（product/pm）
- 技术可行性：架构师（engineering/architect）
- 时间估算：项目经理（project/manager）

## 调用我的前置条件
- 必须有明确的目标描述
- 必须提供约束条件（预算、时间、技术栈）
```

### IDENTITY.md — 身份层

定义能力边界和不处理事项：

```markdown
# Planner 的身份

## 能力清单
- [x] 任务分解与依赖分析
- [x] 风险评估
- [x] 资源估算
- [ ] 代码编写（超出范围）
- [ ] UI 设计（超出范围）

## 不处理事项
- 不写具体代码实现
- 不做视觉设计
- 不替用户做最终决策（只提供选项）
```

**设计洞察**：三文件分离的妙处在于**权限管理**——协作契约（AGENTS.md）可以被系统读取并强制执行，防止 Agent 越权调用不该调用的其他 Agent。

---

## 去中心化协作模型

### agentToAgent：核心协作原语

```python
# Agent A 直接调用 Agent B
result = agent_a.agent_to_agent(
    target_agent_id="coder",
    task="根据以下规划写代码..."
)
```

**契约检查**：调用前系统自动验证：
1. 目标 Agent 是否已注册？
2. 调用者是否在 AGENTS.md 中声明了对目标的依赖？
3. 目标 Agent 当前是否可用？

### 网状通信 vs 星型通信

| 模式 | 代表 | 特点 |
|------|------|------|
| **网状** | OpenClaw | Agent 直接对话，低延迟，但可能产生循环依赖 |
| **星型** | Claude Code | 通过 Orchestrator 中转，可控性强，但 Orchestrator 是瓶颈 |

OpenClaw 选择网状的原因：Agent 之间往往需要频繁、细粒度的协作（如 Coder 和 Reviewer 之间的多轮往返），通过中介转发会大幅增加延迟。

---

## 常驻运行：Cron 驱动的 Agent

OpenClaw 的 Agent 不是"按需唤醒"，而是**常驻内存的后台服务**：

```python
class CronScheduler:
    def run(self):
        while True:
            for agent_id, cron_expr in self.schedules.items():
                if self._should_wake(cron_expr):
                    # 发送唤醒消息（非阻塞）
                    self.bus.send(
                        to=agent_id,
                        message=Message(type="cron_wake")
                    )
            sleep(60)
```

**典型值守场景**：
- 每小时检查一次网站可用性，异常时发 Slack 通知
- 每天早 9 点汇总昨日数据，生成日报发送到钉钉群
- 每周一扫描代码库，提交技术债务报告

这与 Claude Code 的"人类输入触发"形成鲜明对比——OpenClaw 的 Agent 更像是**自动化的 cron job**，只是用自然语言代替了 shell 脚本。

---

## 多平台 Gateway

```python
class Gateway:
    def __init__(self):
        self.adapters = {
            "discord": DiscordAdapter(),
            "slack": SlackAdapter(),
            "telegram": TelegramAdapter(),
            "wechat": WeChatAdapter(),
            # ... 20+ 平台
        }
```

**入站流程**：外部平台消息 → 格式标准化 → 路由到目标 Agent → Agent 处理 → 回复格式化 → 发送到原平台

**关键设计**：每个平台的特性（如 Slack 的线程、Discord 的 Embed、微信的@消息）被封装在 Adapter 中，Agent 本身不需要关心平台差异。

---

## 模型无关设计

OpenClaw 支持 10 种 LLM 提供商，其中 7 种免 API key：

| 提供商 | 接入方式 | 费用 |
|--------|---------|------|
| Gemini CLI | 官方 CLI | 免费（1000 次/天） |
| OpenClaw CLI | 自研 CLI | 免费 |
| Hermes CLI | 开源 CLI | 免费 |
| Ollama | 本地部署 | 免费 |
| Claude Code | 官方 CLI | 需会员 |
| Copilot CLI | 官方 CLI | 需订阅 |
| Codex CLI | 官方 CLI | 需 Plus/Pro |
| DeepSeek | API | 需 key |
| Claude | API | 需 key |
| OpenAI | API | 需 key |

**策略**：通过 CLI 工具调用而非直接调用 API，可以复用用户已有的 AI 订阅，实现"零额外成本"。

---

## 开源生态

OpenClaw 的社区生态非常丰富：

| 项目 | 定位 |
|------|------|
| **openclaw-agents** | 9 个预设 Agent 一键部署 |
| **Fleet** | 多 Agent 舰队管理 CLI |
| **Clawe** | 带 Web Dashboard 的多 Agent 协调系统 |
| **Clawion** | 基于文件的任务协调器 |
| **AionUi** | 统一的 24/7 Cowork 应用 |

---

## 优势与局限

### 优势

- **渠道覆盖最广**：20+ 平台的原生接入
- **成本最低**：7 种免 API key 的接入方式
- **常驻运行**：Cron 调度适合值守场景
- **模型无关**：不被任何单一模型锁定
- **去中心化**：Agent 直接通信，延迟低

### 局限

- **安全责任在用户**：弱隔离、共享状态，需要自行配置安全策略
- **调试困难**：网状通信导致调用链难以追踪
- **上下文管理**：无强制隔离，可能出现信息过载
- **生态成熟度**：相比 Claude Code 的 MCP 生态，工具集成深度不足

---

## 适用场景

| 场景 | 匹配度 | 原因 |
|------|--------|------|
| 7×24 值守监控 | ★★★★★ | Cron + 多平台通知是核心竞争力 |
| 跨团队协作 | ★★★★★ | 直接接入 Slack/Discord/飞书 |
| 多模型实验 | ★★★★★ | 同时跑 GPT-4、Claude、Kimi |
| 代码开发 | ★★★☆☆ | 无 IDE 集成，代码感知弱 |
| 复杂任务分解 | ★★★☆☆ | 无内置 Orchestrator，需自行设计协作流程 |

---

## 参考

- [OpenClaw 开源生态](https://github.com/shenhao-stu/openclaw-agents)
- [Fleet — 多 Agent 舰队管理](https://github.com/oguzhnatly/fleet)
- [Clawe — Web Dashboard 协调系统](https://github.com/getclawe/clawe)
- [Claude Code 与 OpenClaw 架构对比](/research/claude-code-vs-openclaw/)
