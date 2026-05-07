---
title: OpenClaw 记忆层设计
description: OpenClaw 的长期记忆、状态持久化和 Cron 常驻运行机制。
tags: [openclaw, memory, persistence, cron]
---

# OpenClaw 记忆层设计

## 三层记忆模型

```
┌─────────────────────────────────────┐
│  Layer 3: 长期记忆（VectorStore）     │  ← 跨会话持久化知识
│  基于向量检索的历史经验库              │
├─────────────────────────────────────┤
│  Layer 2: 会话状态（Message Thread）  │  ← 当前对话上下文
│  当前 thread 的消息历史                │
├─────────────────────────────────────┤
│  Layer 1: 任务状态（Agent.state）     │  ← 执行跟踪
│  待办列表、执行进度、中间产物          │
└─────────────────────────────────────┘
```

---

## 长期记忆：VectorStore

每个 Agent 拥有独立的向量记忆库：

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

        # 4. 更新记忆
        self.memory.add(f"Task: {task}\nResult: {response}")
        return response
```

**关键设计**：记忆更新是**追加式**的，不会覆盖旧记忆。Agent 在执行每个任务后自动将经验和结果写入记忆库。

---

## 状态持久化：Convex DB + 文件系统

OpenClaw 支持两种持久化后端：

| 后端 | 适用场景 | 特点 |
|------|---------|------|
| **Convex DB** | 生产环境 | 实时同步、自动扩容、支持订阅 |
| **文件系统** | 开发/轻量场景 | 简单、零依赖、易于备份 |

```python
class Agent:
    def persist_state(self):
        """将当前状态写入持久化存储"""
        snapshot = {
            "agent_id": self.agent_id,
            "state": self.state,
            "memory_digest": self.memory.digest(),
            "timestamp": datetime.now().isoformat()
        }
        storage.write(f"agents/{self.agent_id}/state.json", snapshot)
```

---

## 常驻运行：Cron 调度

OpenClaw 的 Agent 不是"按需唤醒"，而是**常驻内存的后台服务**：

```python
class CronScheduler:
    def __init__(self, registry: AgentRegistry):
        self.registry = registry
        self.schedules: dict[str, str] = {}  # agentId -> cron_expr

    def schedule(self, agent_id: str, cron: str):
        self.schedules[agent_id] = cron

    def run(self):
        while True:
            for agent_id, cron_expr in self.schedules.items():
                if self._should_wake(cron_expr):
                    agent = self.registry.agents[agent_id]
                    self.registry.message_bus.send(
                        to=agent_id,
                        message=Message(type="cron_wake")
                    )
            sleep(60)
```

**典型值守场景**：
- 每小时检查网站可用性，异常时发 Slack 通知
- 每天早 9 点汇总昨日数据，生成日报
- 每周一扫描代码库，提交技术债务报告

这与 Claude Code 的"人类输入触发"形成鲜明对比——OpenClaw 的 Agent 更像是**自动化的 cron job**，只是用自然语言代替了 shell 脚本。

---

## 记忆隔离 vs 共享

| 维度 | 隔离（每个 Agent 独立） | 共享（全局记忆库） |
|------|----------------------|------------------|
| **隐私性** | 高 | 低 |
| **协作效率** | 低（需显式传递信息） | 高 |
| **上下文污染** | 低 | 高（无关信息干扰） |
| **实现复杂度** | 低 | 高（需权限控制） |

OpenClaw 默认采用**隔离模式**，但支持通过 `agentToAgent` 显式共享特定信息。
