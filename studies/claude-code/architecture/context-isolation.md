---
title: 上下文隔离机制
permalink: /studies/claude-code/architecture/context-isolation/
---

# 上下文隔离机制

## 核心设计

每个子 Agent 只接收**任务相关的上下文片段**，而非完整会话历史。

```python
class Orchestrator:
    def dispatch(self, user_input: str) -> str:
        required_agents = self._route_intent(user_input)
        results = []
        for agent_id in required_agents:
            agent = self._get_or_create_agent(agent_id)
            # 只传递任务相关的上下文片段
            task_package = {
                "task": user_input,
                "context": self._filter_context(agent_id, self.session_context),
                "tools": agent.allowed_tools
            }
            result = agent.execute(task_package)
            results.append(result)
        return self._synthesize(results)
```

---

## _filter_context 的作用

从完整会话历史中，只提取与当前 Agent 角色相关的信息：

- **后端架构师** → 只接收技术需求、约束条件
- **前端工程师** → 只接收 UI 需求、设计规范
- **测试工程师** → 只接收功能描述、验收标准

**目的**：避免信息过载，减少 token 消耗，提高 Agent 决策质量。

---

## 隔离效果

| 维度 | 隔离前 | 隔离后 |
|------|--------|--------|
| 上下文长度 | 完整会话（可能数千 token） | 相关片段（数百 token） |
| 决策准确率 | 低（信息噪声大） | 高（聚焦相关） |
| token 成本 | 高 | 低 |
| 安全性 | 低（可能泄露敏感信息） | 高（最小权限原则） |
