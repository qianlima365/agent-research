---
title: 动态组队机制
permalink: /studies/openclaw/multi-agent/dynamic-team/
---

# 动态组队机制

## 能力发现

```python
class AgentRegistry:
    def discover(self, capability: str) -> list[str]:
        """根据能力发现 Agent（用于动态组队）"""
        matches = []
        for agent_id, agent in self.agents.items():
            if capability in agent.identity.capabilities:
                matches.append(agent_id)
        return matches
```

---

## 流水线组队

```python
# 发现所有具备相关能力的 Agent
coders = registry.discover("code_writing")
reviewers = registry.discover("code_review")
testers = registry.discover("testing")

# 动态组建流水线
for task in tasks:
    # 编码
    code = registry.agents[coders[0]].execute(task)
    # 审查
    review = registry.agents[reviewers[0]].execute(code)
    # 测试
    test_result = registry.agents[testers[0]].execute(code)
```

---

## 消息类型

| 类型 | 触发方式 | 用途 |
|------|---------|------|
| **direct** | 点对点发送 | 特定 Agent 执行任务 |
| **broadcast** | 广播给所有 Agent | 事件通知、状态同步 |
| **cron_wake** | Cron 定时触发 | 值守任务、周期性检查 |

---

## 网状通信 vs 星型通信

| 维度 | 网状（OpenClaw） | 星型（Claude Code） |
|------|-----------------|-------------------|
| **通信路径** | Agent A ↔ Agent B 直接 | Agent A → Orchestrator → Agent B |
| **延迟** | 低（一跳） | 高（两跳 + 上下文打包） |
| **可控性** | 弱（可能循环依赖） | 强（Orchestrator 统一管控） |
| **适用场景** | 高频细粒度协作 | 低频粗粒度任务分配 |
