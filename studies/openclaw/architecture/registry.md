---
title: Agent Registry 与消息总线
permalink: /studies/openclaw/architecture/registry/
---

# Agent Registry 与消息总线

## Agent Registry

注册中心管理所有活跃 Agent 的生命周期：

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
        return agent

    def discover(self, capability: str) -> list[str]:
        """根据能力发现 Agent（用于动态组队）"""
        matches = []
        for agent_id, agent in self.agents.items():
            if capability in agent.identity.capabilities:
                matches.append(agent_id)
        return matches
```

---

## Message Bus

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

**设计要点**：
- 发送是**异步**的，避免阻塞 Agent 执行
- 支持**广播**模式，用于事件通知和状态同步
- 消息投递失败后会自动重试（指数退避）

---

## 动态发现机制

```python
# 根据能力发现 Agent
coders = registry.discover("code_writing")
reviewers = registry.discover("code_review")

# 动态组建流水线
for coder_id in coders:
    code = registry.agents[coder_id].execute(task)
    for reviewer_id in reviewers:
        review = registry.agents[reviewer_id].execute(code)
```
