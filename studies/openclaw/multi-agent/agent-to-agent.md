---
title: agentToAgent 协作原语
permalink: /studies/openclaw/multi-agent/agent-to-agent/
---

# agentToAgent 协作原语

## 基本用法

```python
# Agent A 直接调用 Agent B
result = agent_a.agent_to_agent(
    target_agent_id="coder",
    task="根据以下规划写代码..."
)
```

---

## 契约检查

调用前系统自动验证三层权限：

1. **存在性检查**：目标 Agent 是否已注册？
2. **依赖声明**：调用者是否在 AGENTS.md 中声明了对目标的依赖？
3. **可用性检查**：目标 Agent 当前是否在线？

```python
def agent_to_agent(self, target_agent_id: str, task: str) -> str:
    if target_agent_id not in registry.agents:
        raise AgentNotFoundError(target_agent_id)

    if target_agent_id not in self.contract.dependencies:
        raise ContractViolationError(
            f"未声明对 {target_agent_id} 的依赖"
        )

    target = registry.agents[target_agent_id]
    return target.execute(task, from_agent=self.agent_id)
```

---

## 调用链追踪

```python
# Agent A 调用 Agent B，Agent B 调用 Agent C
result = agent_a.agent_to_agent("planner", "规划任务")
# 内部调用链：A → planner → architect → ...
```

**调试支持**：每次 `agentToAgent` 调用都会记录调用链，便于追踪问题根源。
