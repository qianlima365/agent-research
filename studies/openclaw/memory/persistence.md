---
title: 状态持久化机制
permalink: /studies/openclaw/memory/persistence/
---

# 状态持久化机制

## 持久化后端

OpenClaw 支持两种持久化后端：

| 后端 | 适用场景 | 特点 |
|------|---------|------|
| **Convex DB** | 生产环境 | 实时同步、自动扩容、支持订阅 |
| **文件系统** | 开发/轻量场景 | 简单、零依赖、易于备份 |

---

## 快照机制

```python
class Agent:
    def persist_state(self):
        snapshot = {
            "agent_id": self.agent_id,
            "state": self.state,
            "memory_digest": self.memory.digest(),
            "timestamp": datetime.now().isoformat()
        }
        storage.write(f"agents/{self.agent_id}/state.json", snapshot)
```

**触发时机**：
- 每个任务执行完成后
- 定时快照（每 5 分钟）
- Agent 优雅关闭时

---

## 恢复流程

```python
def restore_agent(agent_id: str) -> Agent:
    snapshot = storage.read(f"agents/{agent_id}/state.json")
    agent = Agent(agent_id=snapshot["agent_id"])
    agent.state = snapshot["state"]
    agent.memory.restore(snapshot["memory_digest"])
    return agent
```

**应用场景**：
- 服务重启后恢复 Agent 状态
- 将 Agent 状态迁移到另一台服务器
- 版本回滚时恢复到历史状态
