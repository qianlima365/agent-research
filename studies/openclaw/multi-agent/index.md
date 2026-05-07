---
title: OpenClaw 多智能体设计
description: OpenClaw 的去中心化多智能体协作模型，包括 agentToAgent 原语、网状通信和动态组队。
tags: [openclaw, multi-agent, decentralized, collaboration]
---

# OpenClaw 多智能体设计

## 核心协作原语：agentToAgent

```python
# Agent A 直接调用 Agent B
result = agent_a.agent_to_agent(
    target_agent_id="coder",
    task="根据以下规划写代码..."
)
```

**契约检查**（调用前自动验证）：
1. 目标 Agent 是否已注册？
2. 调用者是否在 AGENTS.md 中声明了对目标的依赖？
3. 目标 Agent 当前是否可用？

```python
def agent_to_agent(self, target_agent_id: str, task: str) -> str:
    if target_agent_id not in registry.agents:
        raise AgentNotFoundError(target_agent_id)

    # 检查契约权限
    if target_agent_id not in self.contract.dependencies:
        raise ContractViolationError(
            f"未声明对 {target_agent_id} 的依赖"
        )

    target = registry.agents[target_agent_id]
    return target.execute(task, from_agent=self.agent_id)
```

---

## 网状通信 vs 星型通信

| 维度 | 网状（OpenClaw） | 星型（Claude Code） |
|------|-----------------|-------------------|
| **通信路径** | Agent A ↔ Agent B 直接 | Agent A → Orchestrator → Agent B |
| **延迟** | 低（一跳） | 高（两跳 + 上下文打包） |
| **可控性** | 弱（可能循环依赖） | 强（Orchestrator 统一管控） |
| **调试难度** | 高（调用链分散） | 低（集中日志） |
| **适用场景** | 高频细粒度协作 | 低频粗粒度任务分配 |

**OpenClaw 选择网状的原因**：Agent 之间往往需要频繁往返协作（如 Coder 和 Reviewer 的多轮迭代），通过中介转发会大幅增加延迟。

---

## 动态组队

```python
class AgentRegistry:
    def discover(self, capability: str) -> list[str]:
        """根据能力发现 Agent（用于动态组队）"""
        matches = []
        for agent_id, agent in self.agents.items():
            if capability in agent.identity.capabilities:
                matches.append(agent_id)
        return matches

# 使用示例
coders = registry.discover("code_writing")
reviewers = registry.discover("code_review")

# 动态组建流水线
for coder_id in coders:
    code = registry.agents[coder_id].execute(task)
    for reviewer_id in reviewers:
        review = registry.agents[reviewer_id].execute(code)
```

---

## 消息类型

| 类型 | 触发方式 | 用途 |
|------|---------|------|
| **direct** | 点对点发送 | 特定 Agent 执行任务 |
| **broadcast** | 广播给所有 Agent | 事件通知、状态同步 |
| **cron_wake** | Cron 定时触发 | 值守任务、周期性检查 |

---

## 与 Claude Code 协作模型的对比

| 维度 | OpenClaw | Claude Code |
|------|---------|------------|
| **架构** | 去中心化网状 | 中心化星型 |
| **通信** | Agent 直接对话 | 通过 Orchestrator 中转 |
| **权限** | 契约声明 + 运行时检查 | Orchestrator 集中管控 |
| **隔离** | 弱（共享文件/DB） | 强（独立上下文窗口） |
| **调度** | Cron + 事件混合 | 事件驱动（用户输入） |
| **错误恢复** | Agent 自主处理 + 状态回滚 | Orchestrator 重试/降级 |

---

## Related Notes

{% assign current_dir = page.path | split: '/' | pop | join: '/' %}
{% for doc in site.studies %}
  {% assign doc_dir = doc.path | split: '/' | pop | join: '/' %}
  {% if doc_dir == current_dir and doc.name != 'index.md' %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
  {% endif %}
{% endfor %}
