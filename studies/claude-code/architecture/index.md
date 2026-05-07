---
title: Claude Code 基础架构
description: Claude Code 的 Orchestrator-Worker 分层隔离架构，包括四层分工、上下文隔离和权限模型。
tags: [claude-code, architecture, orchestrator, isolation]
---

# Claude Code 基础架构

## 四层隔离架构

```
┌─────────────────────────────────────────────┐
│  Layer 4: Orchestrator（主会话调度器）         │
│  - 解析用户意图，决定调用哪个子 Agent           │
│  - 管理子 Agent 的生命周期                     │
│  - 所有子 Agent 的输出汇总后返回给用户          │
├─────────────────────────────────────────────┤
│  Layer 3: SubAgent Pool（子 Agent 池）         │
│  - 每个子 Agent 有独立的 Context Window         │
│  - 权限分级：只读(default) / 读写 / 危险操作    │
│  - 子 Agent 之间不可直接通信                    │
├─────────────────────────────────────────────┤
│  Layer 2: MCP Client（工具中间层）              │
│  - 统一协议与外部工具交互                       │
│  - 工具调用前的权限检查                         │
│  - 结果序列化后注入子 Agent 上下文              │
├─────────────────────────────────────────────┤
│  Layer 1: Tool Ecosystem（工具生态）            │
│  - FileSystem / Git / Browser / Database ...   │
│  - 自定义 MCP Server                           │
└─────────────────────────────────────────────┘
```

---

## Orchestrator-Worker 模型

### 星型拓扑

```
    ┌─────────────────────────┐
    │    主 Claude 会话        │  ← Orchestrator
    └──────┬────────┬─────────┘
           │        │
     ┌─────┘        └─────┐
     ↓                    ↓
┌─────────┐         ┌─────────┐
│ Agent A │         │ Agent B │  ← Worker（隔离上下文）
│ 只读     │         │ 读写    │
└─────────┘         └─────────┘
```

**关键约束**：
- Agent 之间**不能直接对话**，所有通信必须通过 Orchestrator 中转
- 每个 Worker 只接收**任务相关的上下文片段**
- 默认权限为**只读**

---

## 权限模型

| 操作类型 | 默认权限 | 需要确认 |
|---------|---------|---------|
| 读取文件 | ✅ 允许 | 否 |
| 写入文件 | ⚠️ 需授权 | 是 |
| 执行命令 | ⚠️ 需授权 | 是 |
| 删除文件 | ❌ 禁止 | 必须人类确认 |
| 网络请求 | ⚠️ 需授权 | 是 |

**设计理由**：在代码开发场景中，一个失控的 Agent 可能误删文件、引入安全漏洞。Orchestrator 作为"内核"，严格控制每个子进程的权限。

---

## 上下文隔离机制

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

**_filter_context 的作用**：从完整会话历史中，只提取与当前 Agent 角色相关的信息，避免信息过载。

---

## 状态与记忆

| 层级 | 实现 | 作用 |
|------|------|------|
| **短期记忆** | 各 Agent 独立上下文窗口 | 单次任务执行 |
| **中期记忆** | `CLAUDE.md` 文件 | 项目级共享上下文，跨会话 |
| **长期记忆** | MCP 向量数据库 | 持久化知识库 |

---

## Related Notes

{% assign current_dir = page.path | split: '/' | pop | join: '/' %}
{% for doc in site.studies %}
  {% assign doc_dir = doc.path | split: '/' | pop | join: '/' %}
  {% if doc_dir == current_dir and doc.name != 'index.md' %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
  {% endif %}
{% endfor %}
