---
title: OpenClaw 基础架构
description: OpenClaw 的注册中心 + 消息总线 + 多平台 Gateway 三层架构详解。
tags: [openclaw, architecture, gateway, registry]
---

# OpenClaw 基础架构

## 三层架构概览

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

---

## Layer 3: Gateway

Gateway 是 OpenClaw 与外部世界交互的统一入口。

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

    def on_inbound(self, platform: str, raw_message: dict):
        # 1. 格式标准化
        msg = self.adapters[platform].normalize(raw_message)
        # 2. 路由到目标 Agent
        target_agent = self._route(msg)
        # 3. 转发到 Agent Registry
        registry.message_bus.send(
            to=target_agent,
            message=Message(type="direct", content=msg.text)
        )
```

**关键设计**：各平台特性（Slack 线程、Discord Embed、微信@消息）被封装在 Adapter 中，Agent 本身不感知平台差异。

---

## Layer 2: Agent Registry + Message Bus

### Agent 注册

```python
class AgentRegistry:
    def __init__(self):
        self.agents: dict[str, Agent] = {}
        self.message_bus = MessageBus()

    def register(self, workspace_path: str):
        agent_id = Path(workspace_path).name
        soul = load_markdown(f"{workspace_path}/SOUL.md")
        contract = load_markdown(f"{workspace_path}/AGENTS.md")
        identity = load_markdown(f"{workspace_path}/IDENTITY.md")

        agent = Agent(agent_id=agent_id, soul=soul,
                      contract=contract, identity=identity,
                      llm=create_llm(identity.model_preference))

        self.agents[agent_id] = agent
        self.message_bus.subscribe(agent_id, agent.on_message)
```

### 消息总线

```python
class MessageBus:
    def send(self, to: str, message: Message):
        # 异步投递，不阻塞发送方
        asyncio.create_task(self._deliver(to, message))

    def broadcast(self, message: Message):
        # 广播给所有 Agent
        for agent_id in registry.agents:
            self.send(agent_id, message)
```

---

## Layer 1: Agent Workers

Agent Worker 是实际执行业务逻辑的单元：

```python
class Agent:
    def __init__(self, agent_id, soul, contract, identity, llm):
        self.agent_id = agent_id
        self.soul = soul              # 核心人格
        self.contract = contract      # 协作契约
        self.identity = identity      # 身份边界
        self.llm = llm
        self.state = {}               # 运行时状态
        self.memory = VectorStore()   # 长期记忆

    def on_message(self, message: Message):
        if message.type == "direct":
            result = self.execute(message.content)
            self.send_reply(message.from_agent, result)
        elif message.type == "cron_wake":
            pending = self._check_pending_tasks()
            for task in pending:
                self.execute(task)
```

---

## 选型对比

| 维度 | OpenClaw 网关 | 自建 WebSocket | 各平台原生 SDK |
|------|--------------|---------------|--------------|
| 接入成本 | 低（统一接口） | 中 | 高（需适配每个平台） |
| 功能完整度 | 中（通用抽象） | 高（完全控制） | 高（原生能力） |
| 维护成本 | 低（社区维护 Adapter） | 高 | 高（需跟进各平台更新） |

---

## Related Notes

{% assign current_dir = page.path | split: '/' | pop | join: '/' %}
{% for doc in site.studies %}
  {% assign doc_dir = doc.path | split: '/' | pop | join: '/' %}
  {% if doc_dir == current_dir and doc.name != 'index.md' %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
  {% endif %}
{% endfor %}
