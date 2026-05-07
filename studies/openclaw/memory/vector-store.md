---
title: VectorStore 长期记忆
permalink: /studies/openclaw/memory/vector-store/
---

# VectorStore 长期记忆

## 记忆模型

每个 Agent 拥有独立的向量记忆库，用于跨会话持久化知识和经验。

---

## 读写流程

```python
class Agent:
    def __init__(self, ...):
        self.memory = VectorStore()   # 长期记忆

    def execute(self, task: str):
        # 1. 检索相关记忆
        relevant = self.memory.search(task, top_k=3)

        # 2. 构建提示（注入记忆）
        prompt = f"""
        {self.soul.core_values}
        相关历史记忆：{relevant}
        当前任务：{task}
        """

        # 3. 执行
        response = self.llm.generate(prompt)

        # 4. 更新记忆（追加式）
        self.memory.add(f"Task: {task}\nResult: {response}")
        return response
```

---

## 关键设计

| 设计 | 说明 |
|------|------|
| **追加式更新** | 新经验不覆盖旧记忆，避免遗忘 |
| **自动索引** | 每次 add 后自动构建向量索引 |
| **语义检索** | 基于 embedding 相似度，而非关键词匹配 |
| **隔离性** | 每个 Agent 有独立的记忆库，默认不共享 |

---

## 记忆隔离 vs 共享

| 维度 | 隔离（每个 Agent 独立） | 共享（全局记忆库） |
|------|----------------------|------------------|
| **隐私性** | 高 | 低 |
| **协作效率** | 低（需显式传递信息） | 高 |
| **上下文污染** | 低 | 高（无关信息干扰） |
| **实现复杂度** | 低 | 高（需权限控制） |

OpenClaw 默认采用**隔离模式**，但支持通过 `agentToAgent` 显式共享特定信息。
