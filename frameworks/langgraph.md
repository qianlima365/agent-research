---
title: LangGraph
date: 2026-05-07
description: LangChain 生态的工作流编排层，基于状态图模型实现复杂多智能体协作，原生支持循环、条件分支、状态持久化和人机协作断点。
tags: [framework, langchain, state-machine, workflow, multi-agent]
---

# LangGraph

**官网**: [langchain-ai.github.io/langgraph](https://langchain-ai.github.io/langgraph/) | **GitHub**: [langchain-ai/langgraph](https://github.com/langchain-ai/langgraph)

> LangChain 生态的"流程编排层"，用状态机模型解决复杂多智能体工作流中的循环、条件分支和状态持久化问题。

---

## 概述

LangGraph 是 LangChain 团队在 2024 年初推出的工作流编排框架。它的诞生源于一个观察：**LCEL（LangChain Expression Language）擅长线性管道组合，但无法优雅地表达复杂工作流中的循环、条件分支和状态共享**。

LangGraph 的核心抽象是 **StateGraph（状态图）**：
- **节点（Node）** = 执行单元（函数或 Agent）
- **边（Edge）** = 状态流转规则
- **状态（State）** = 全局共享的数据结构，所有节点读写同一个状态对象

这种设计与传统"链式调用"的本质区别在于：**LangGraph 允许数据流反向或循环，而不仅是从左到右的线性传递**。

---

## 核心概念

### 1. 状态（State）

状态是 LangGraph 的全局数据容器，所有节点通过同一个状态对象交换信息：

```python
from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

class AgentState(TypedDict):
    """全局状态定义"""
    messages: Annotated[list, add_messages]  # 对话历史（自动累加）
    plan: str                                # 规划结果
    code: str                                # 代码输出
    review: str                              # 审查意见
    approved: bool                           # 是否通过
    iteration: int                           # 迭代计数
```

**关键机制：Reducer**

`Annotated[list, add_messages]` 中的 `add_messages` 是一个 **Reducer 函数**。它告诉 LangGraph：当多个节点同时向 `messages` 字段写入时，应该如何合并结果（这里是追加消息，而不是覆盖）。

```python
# 自定义 Reducer：数字累加
def add_numbers(a: int, b: int) -> int:
    return a + b

class State(TypedDict):
    total: Annotated[int, add_numbers]  # 多个节点写入时自动求和
```

### 2. 节点（Node）

节点是一个接收状态、返回状态更新的函数：

```python
def planner_node(state: AgentState):
    """规划节点：将需求分解为任务"""
    response = llm.invoke([
        ("system", "你是任务规划专家"),
        ("human", state["requirement"])
    ])
    return {
        "plan": response.content,
        "messages": [("assistant", response.content)],
        "iteration": 0
    }
```

**设计原则**：
- 节点函数只返回**状态更新**（delta），不是完整状态
- LangGraph 自动将 delta 合并到全局状态中
- 节点之间不直接通信，只能通过状态间接交换信息

### 3. 边（Edge）

#### 普通边：固定流转

```python
workflow.add_edge("planner", "coder")      # planner 之后一定到 coder
workflow.add_edge("coder", "reviewer")     # coder 之后一定到 reviewer
```

#### 条件边：动态路由

```python
def should_continue(state: AgentState) -> str:
    if state["approved"]:
        return END          # 通过审查，结束
    if state["iteration"] >= 3:
        return END          # 超过最大迭代，结束
    return "coder"          # 未通过，回到 coder 重写

# reviewer 节点之后，根据 should_continue 的返回值决定去向
workflow.add_conditional_edges("reviewer", should_continue)
```

**条件边的返回值** 必须是目标节点的名称或 `END`（内置的结束常量）。

---

## 完整工作流示例

### 场景：代码生成 + 自动迭代优化

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")

# 1. 定义状态
class AgentState(TypedDict):
    requirement: str
    messages: Annotated[list, add_messages]
    plan: str
    code: str
    review: str
    approved: bool
    iteration: int

# 2. 定义节点
def planner_node(state: AgentState):
    response = llm.invoke([
        ("system", "你是任务规划专家。将需求分解为步骤。"),
        ("human", state["requirement"])
    ])
    return {
        "plan": response.content,
        "messages": [("assistant", response.content)],
        "iteration": 0
    }

def coder_node(state: AgentState):
    prompt = f"规划：{state['plan']}\n"
    if state.get("review"):
        prompt += f"\n上一轮审查意见：{state['review']}\n请修改。"

    response = llm.invoke([
        ("system", "你是 Python 专家。写出高质量代码。"),
        ("human", prompt)
    ])
    return {
        "code": response.content,
        "messages": [("assistant", response.content)],
        "iteration": state["iteration"] + 1
    }

def reviewer_node(state: AgentState):
    response = llm.invoke([
        ("system", "你是代码审查专家。如果合格，最后一行输出 APPROVED。"),
        ("human", f"代码：\n{state['code']}")
    ])
    return {
        "review": response.content,
        "approved": "APPROVED" in response.content.upper(),
        "messages": [("assistant", response.content)]
    }

# 3. 定义条件路由
def should_continue(state: AgentState) -> str:
    if state["approved"]:
        return END
    if state["iteration"] >= 3:
        return END
    return "coder"

# 4. 构建图
workflow = StateGraph(AgentState)

workflow.add_node("planner", planner_node)
workflow.add_node("coder", coder_node)
workflow.add_node("reviewer", reviewer_node)

workflow.set_entry_point("planner")
workflow.add_edge("planner", "coder")
workflow.add_edge("coder", "reviewer")
workflow.add_conditional_edges("reviewer", should_continue)

# 5. 编译并运行
graph = workflow.compile()

result = graph.invoke({"requirement": "写带缓存的 Fibonacci 函数"})
print(f"迭代 {result['iteration']} 次，通过：{result['approved']}")
print(result["code"])
```

**执行流程**：

```
requirement
    ↓
planner ──> plan
    ↓
coder ──> code
    ↓
reviewer ──> review, approved=false
    ↓（条件：未通过，回到 coder）
coder ──> code（带 review 反馈）
    ↓
reviewer ──> review, approved=true
    ↓（条件：通过，END）
输出结果
```

---

## 与 LangChain LCEL 的关系

| 维度 | LCEL | LangGraph |
|------|------|-----------|
| **数据流** | 线性管道（A → B → C） | 任意图结构（可循环、分支） |
| **状态** | 无全局状态，每步输出是下一步输入 | 有全局状态，所有节点共享 |
| **循环** | 需借助外部 `while` | 原生支持（条件边返回自身） |
| **适用** | 简单串并联、确定性流程 | 复杂工作流、需要迭代优化 |
| **代码风格** | 声明式管道 `A | B | C` | 命令式建图 `add_node` + `add_edge` |

**最佳实践：两者结合**

```python
from langchain_core.runnables import RunnableLambda

# LCEL 封装单个 Agent 的内部逻辑（复用、测试）
planner_runnable = RunnableLambda(lambda x: llm.invoke(f"规划：{x}").content)
coder_runnable = RunnableLambda(lambda x: llm.invoke(f"编码：{x}").content)

# LangGraph 编排多个 Agent 的协作关系
def planner_node(state):
    return {"plan": planner_runnable.invoke(state["requirement"])}

def coder_node(state):
    return {"code": coder_runnable.invoke(state["plan"])}

workflow = StateGraph(AgentState)
workflow.add_node("planner", planner_node)
workflow.add_node("coder", coder_node)
workflow.add_edge("planner", "coder")
graph = workflow.compile()
```

**分工**：LCEL 负责"组件内部怎么干"，LangGraph 负责"组件之间怎么配合"。

---

## 状态持久化（Checkpointer）

LangGraph 内置持久化机制，可以在任意节点暂停、保存状态、之后恢复：

```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.checkpoint.sqlite import SqliteSaver

# 内存模式（开发测试）
graph = workflow.compile(checkpointer=MemorySaver())

# SQLite 模式（生产推荐）
graph = workflow.compile(checkpointer=SqliteSaver("./checkpoints.db"))

# Redis 模式（分布式场景）
# graph = workflow.compile(checkpointer=RedisSaver(...))
```

**关键能力**：
- **断点续跑**：任务中断后从上次完成的节点恢复
- **时间旅行**：回溯到任意历史状态重新执行
- **多会话隔离**：`thread_id` 区分不同用户/任务的独立状态

```python
config = {"configurable": {"thread_id": "user-123"}}

# 第一次执行
result = graph.invoke({"requirement": "写 Fibonacci"}, config)

# 之后可以从检查点恢复，继续对话
result2 = graph.invoke({"requirement": "再加个缓存"}, config)
```

---

## 人机协作（Human-in-the-loop）

```python
# 在特定节点前暂停，等待人类输入
graph = workflow.compile(
    checkpointer=MemorySaver(),
    interrupt_before=["reviewer"]  # 在 reviewer 之前中断
)

config = {"configurable": {"thread_id": "session-001"}}

# 运行到 reviewer 之前暂停
for event in graph.stream(
    {"requirement": "写 Fibonacci"},
    config,
    stream_mode="values"
):
    print(event)

# === 此时流程暂停，人类可以审查 code ===

# 人类确认后继续执行
for event in graph.stream(None, config, stream_mode="values"):
    print(event)
```

**典型场景**：
- 代码生成后人类审查再执行
- 数据分析结果人工确认后再发送报告
- 敏感操作（转账、删除数据）前人工审批

---

## 多智能体协作模式

### 模式一：监督者（Supervisor）

```
         ┌─────────────┐
         │  Supervisor │  ← 分配任务、汇总结果
         └──────┬──────┘
    ┌───────────┼───────────┐
    ↓           ↓           ↓
┌───────┐  ┌───────┐  ┌───────┐
│Research│  │ Coder │  │ Test  │
└───┬───┘  └───┬───┘  └───┬───┘
    └───────────┴───────────┘
                ↓
         ┌─────────────┐
         │  Supervisor │  ← 再次调度
         └─────────────┘
```

```python
# Supervisor 节点决定下一个执行哪个 Agent
def supervisor_node(state):
    members = ["researcher", "coder", "tester"]
    response = llm.invoke(f"选择下一个执行者：{members}")
    return {"next": response.content}

workflow.add_conditional_edges(
    "supervisor",
    lambda x: x["next"],
    {"researcher": "researcher", "coder": "coder", "tester": "tester", "END": END}
)
```

### 模式二：辩论（Debate）

```python
# 两个 Agent 交替发言，直到达成共识
def agent_a_node(state):
    return {"messages": [("assistant", llm_a.invoke(state["topic"]).content)]}

def agent_b_node(state):
    return {"messages": [("assistant", llm_b.invoke(state["topic"]).content)]}

def consensus_check(state):
    if "AGREE" in state["messages"][-1].content:
        return END
    return "agent_a"  # 继续辩论

workflow.add_conditional_edges("agent_b", consensus_check)
```

---

## 优势与局限

### 优势

| 优势 | 说明 |
|------|------|
| **循环原生支持** | 不需要外部 `while`，条件边直接返回自身即可 |
| **状态可视化** | `graph.get_graph().draw_mermaid_png()` 生成流程图 |
| **持久化内置** | Checkpointer 机制比手写状态管理更可靠 |
| **人机断点** | `interrupt_before`/`interrupt_after` 是杀手级功能 |
| **与 LangChain 生态无缝** | 所有 LCEL Runnable 都可以直接作为节点 |

### 局限

| 局限 | 说明 |
|------|------|
| **学习门槛** | 需要理解状态机、Reducer、TypedDict 等概念 |
| **调试复杂** | 循环工作流的中间状态追踪比线性链困难 |
| **性能敏感场景** | 状态序列化/反序列化有开销，极端性能场景需优化 |
| **图结构约束** | 必须是 DAG（有向无环图），除非用条件边实现循环 |

---

## 适用场景

| 场景 | 为什么适合 LangGraph |
|------|---------------------|
| **代码审查迭代** | 编码 → 审查 →（未通过）→ 重写，天然循环 |
| **多轮对话 Agent** | 用户输入 → 推理 → 工具调用 → 观察 → 再推理 |
| **审批工作流** | 自动处理 → 人工审批 →（通过/拒绝）→ 不同分支 |
| **多 Agent 竞争** | 多个 Agent 并行产出 → 评审 → 最优方案 |
| **错误恢复** | 某步骤失败 → 重试 →（超过次数）→ 降级处理 |

---

## 版本与安装

```bash
# Python
pip install langgraph

# 需要持久化
pip install langgraph-checkpoint-sqlite

# 与 LangChain 一起
pip install langchain langchain-openai langgraph
```

---

## 参考

- [LangGraph 官方文档](https://langchain-ai.github.io/langgraph/)
- [LangGraph 概念指南](https://langchain-ai.github.io/langgraph/concepts/)
- [状态持久化指南](https://langchain-ai.github.io/langgraph/concepts/persistence/)
- [Human-in-the-loop](https://langchain-ai.github.io/langgraph/concepts/human_in_the_loop/)
- [LangChain 框架研究](frameworks/langchain.md)
- [LangChain 与 LangGraph 多智能体实现](development/langchain-vs-langgraph-multi-agent.md)
