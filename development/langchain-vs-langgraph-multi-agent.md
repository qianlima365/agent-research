---
title: 用 LangChain 与 LangGraph 搭建多智能体系统
date: 2026-05-07
---

# 用 LangChain 与 LangGraph 搭建多智能体系统

> LangChain 解决"怎么连"，LangGraph 解决"怎么流"。两者不是替代关系，而是互补。

---

## 定位差异

| 维度 | LangChain (LCEL) | LangGraph |
|------|-------------------|-----------|
| **核心抽象** | Runnable（可运行单元） | StateGraph（状态图） |
| **数据流** | 函数式管道（Functional Pipeline） | 状态机（State Machine） |
| **循环支持** | 需借助外部循环（如 `while`） | 原生支持（图内跳转） |
| **条件分支** | 需借助 `RunnableBranch` | 原生支持（条件边） |
| **状态持久化** | 手动管理 | 内置 Checkpointer |
| **适用场景** | 线性流水线、简单串并联 | 复杂工作流、需要循环迭代 |

**一句话理解**：LCEL 是"管道工"，负责把各个组件按顺序/并行连接起来；LangGraph 是"流程设计师"，负责定义状态如何在不同节点间流转。

---

## LangChain + LCEL：流水线式多 Agent

LCEL（LangChain Expression Language）的核心是 `Runnable` 接口。所有组件（LLM、Prompt、Tool、甚至另一个 Chain）都实现了 `Runnable`，可以用 `|` 运算符像管道一样串联。

### 场景：代码生成流水线

Planner → Coder → Reviewer，串行执行，每步输出作为下一步输入。

```python
from langchain_core.runnables import RunnablePassthrough, RunnableLambda
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")

# 定义三个 Specialist Agent（通过 system prompt 区分角色）
planner = llm.with_config({"run_name": "planner"}).bind(
    system_message="""你是一个任务规划专家。
将用户需求分解为具体的实现步骤，用编号列表输出。
不要写代码，只输出规划。"""
)

coder = llm.with_config({"run_name": "coder"}).bind(
    system_message="""你是一个 Python 专家。
根据给定的规划写出高质量代码。
代码必须包含类型注解、docstring 和异常处理。"""
)

reviewer = llm.with_config({"run_name": "reviewer"}).bind(
    system_message="""你是一个代码审查专家。
检查代码的潜在问题（bug、性能、安全、可读性）。
如果代码合格，在最后一行输出：APPROVED
否则输出具体修改建议。"""
)

# LCEL 管道：数据流从左到右
chain = (
    {"requirement": RunnablePassthrough()}
    | RunnableLambda(lambda x: {"plan": planner.invoke(x["requirement"]).content})
    | RunnableLambda(lambda x: {
        "plan": x["plan"],
        "code": coder.invoke(f"规划：\n{x['plan']}").content
    })
    | RunnableLambda(lambda x: {
        "plan": x["plan"],
        "code": x["code"],
        "review": reviewer.invoke(f"代码：\n{x['code']}").content
    })
)

result = chain.invoke("写一个带缓存的 Fibonacci 计算函数")
print(result["plan"])
print(result["code"])
print(result["review"])
```

### LCEL 的并行能力：RunnableParallel

如果 Planner 分解出多个独立子任务，可以用 `RunnableParallel` 让多个 Coder 同时工作：

```python
from langchain_core.runnables import RunnableParallel

# 假设 planner 输出 "1. 实现核心算法\n2. 实现缓存装饰器\n3. 写单元测试"
# 三个子任务可以并行执行
parallel_chain = RunnableParallel(
    core=lambda x: coder.invoke(f"任务：实现核心算法\n规划：{x['plan']}").content,
    cache=lambda x: coder.invoke(f"任务：实现缓存装饰器\n规划：{x['plan']}").content,
    tests=lambda x: coder.invoke(f"任务：写单元测试\n规划：{x['plan']}").content,
)

chain = (
    {"requirement": RunnablePassthrough()}
    | RunnableLambda(lambda x: {"plan": planner.invoke(x["requirement"]).content})
    | parallel_chain
)

result = chain.invoke("写一个带缓存的 Fibonacci 计算函数")
# result = {"core": "...", "cache": "...", "tests": "..."}
```

**LCEL 的优势**：代码简洁、数据流清晰、天然支持流式输出（`chain.stream()`）。

**LCEL 的局限**：循环、条件分支需要借助外部 Python 逻辑，图的可视化困难。

---

## LangGraph：状态图式多 Agent

LangGraph 把多 Agent 协作建模为**状态图**：节点是 Agent（或函数），边是状态流转规则。关键优势是原生支持循环和条件分支。

### 场景：代码生成 + 自动迭代优化

Planner → Coder → Reviewer →（如果未通过）→ 回到 Coder，最多迭代 3 次。

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, END
from langgraph.graph.message import add_messages
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4")

# 1. 定义全局状态结构
class AgentState(TypedDict):
    requirement: str           # 原始需求
    messages: Annotated[list, add_messages]  # 对话历史（自动累加）
    plan: str                  # 规划结果
    code: str                  # 代码结果
    review: str                # 审查结果
    approved: bool             # 是否通过审查
    iteration: int             # 当前迭代次数

# 2. 定义节点函数（每个节点接收状态，返回状态更新）
def planner_node(state: AgentState):
    """规划节点：将需求分解为任务"""
    response = llm.invoke([
        ("system", "你是一个任务规划专家。将需求分解为具体步骤，不要写代码。"),
        ("human", state["requirement"])
    ])
    return {
        "plan": response.content,
        "messages": [("assistant", response.content)],
        "iteration": 0
    }

def coder_node(state: AgentState):
    """编码节点：根据规划写代码"""
    response = llm.invoke([
        ("system", "你是一个 Python 专家。根据规划写出高质量代码。"),
        ("human", f"规划：\n{state['plan']}\n\n上一轮审查意见（如果有）：\n{state.get('review', '无')}")
    ])
    return {
        "code": response.content,
        "messages": [("assistant", response.content)],
        "iteration": state["iteration"] + 1
    }

def reviewer_node(state: AgentState):
    """审查节点：检查代码质量"""
    response = llm.invoke([
        ("system", "你是一个代码审查专家。如果代码合格，最后一行输出 APPROVED。"),
        ("human", f"代码：\n{state['code']}")
    ])
    approved = "APPROVED" in response.content.upper()
    return {
        "review": response.content,
        "approved": approved,
        "messages": [("assistant", response.content)]
    }

# 3. 定义条件边：判断是否继续迭代
def should_continue(state: AgentState) -> str:
    if state["approved"]:
        return END          # 通过审查，结束
    if state["iteration"] >= 3:
        return END          # 超过最大迭代次数，结束
    return "coder"          # 未通过，回到 coder 节点重写

# 4. 构建状态图
workflow = StateGraph(AgentState)

workflow.add_node("planner", planner_node)
workflow.add_node("coder", coder_node)
workflow.add_node("reviewer", reviewer_node)

workflow.set_entry_point("planner")
workflow.add_edge("planner", "coder")
workflow.add_edge("coder", "reviewer")

# 条件边：reviewer 之后，根据 should_continue 决定去向
workflow.add_conditional_edges("reviewer", should_continue)

# 5. 编译并运行
graph = workflow.compile()

result = graph.invoke({
    "requirement": "写一个带缓存的 Fibonacci 计算函数"
})

print(f"迭代次数：{result['iteration']}")
print(f"是否通过：{result['approved']}")
print(f"最终代码：\n{result['code']}")
print(f"审查意见：\n{result['review']}")
```

### 可视化图结构

```python
from IPython.display import Image, display

# 生成 Mermaid 图（LangGraph 内置支持）
display(Image(graph.get_graph().draw_mermaid_png()))
```

生成的图结构：

```
[Start] → planner → coder → reviewer
                        ↑__________|
                    (未通过则循环)
```

### 状态持久化：人机协作的关键

LangGraph 内置 `Checkpointer`，可以在任意节点暂停，等待人类输入，然后恢复：

```python
from langgraph.checkpoint.memory import MemorySaver

# 添加内存检查点
graph = workflow.compile(checkpointer=MemorySaver())

# 运行到某个节点后暂停（配置 interrupt_before）
config = {"configurable": {"thread_id": "session-001"}}

# 第一次运行，在 reviewer 之前暂停
for event in graph.stream(
    {"requirement": "写 Fibonacci 函数"},
    config,
    interrupt_before=["reviewer"]  # 在 reviewer 节点前中断
):
    print(event)

# 人类审查代码后，决定是否继续
# 从检查点恢复执行
for event in graph.stream(None, config):
    print(event)
```

这实现了**人类在环（Human-in-the-loop）**：Agent 干活 → 人类审查 → Agent 继续，循环往复。

---

## 核心差异总结

| 对比项 | LangChain LCEL | LangGraph |
|--------|----------------|-----------|
| **代码风格** | 管道运算符 `\|` 链式调用 | 显式定义节点和边 |
| **循环实现** | 外部 Python `while` 循环 | 图内 `add_conditional_edges` |
| **状态访问** | 每个步骤只拿到上一步输出 | 全局状态，任意节点可读取全部历史 |
| **断点调试** | 难（链中间状态不透明） | 易（`interrupt_before` 精确暂停） |
| **可视化** | 无内置支持 | 内置 Mermaid 图生成 |
| **流式输出** | `chain.stream()` 原生支持 | `graph.stream()` 支持，但需处理节点边界 |
| **学习曲线** | 平缓（会写函数就会写链） | 稍陡（需理解状态机概念） |

---

## 选型建议

| 你的场景 | 推荐方案 | 理由 |
|---------|---------|------|
| 简单的串行/并行流水线 | **LCEL** | 代码简洁，几行搞定 |
| 需要循环迭代（如自动修复） | **LangGraph** | 原生循环，无需外部 `while` |
| 需要人类在环审批 | **LangGraph** | `interrupt_before`/`interrupt_after` 是杀手级功能 |
| 需要可视化展示流程 | **LangGraph** | 内置 Mermaid 图，方便与非技术人员沟通 |
| 需要持久化状态（断点续跑） | **LangGraph** | `Checkpointer` 支持内存/SQLite/Redis |
| 已有大量 LangChain 组件 | **两者结合** | LCEL 写节点内部逻辑，LangGraph 编排节点流转 |

---

## 混合模式：LCEL 写节点，LangGraph 编排

实际项目中，最佳实践是两者结合：用 LCEL 的 `Runnable` 封装每个 Agent 的内部逻辑，用 LangGraph 的 `StateGraph` 管理它们之间的协作关系。

```python
from langchain_core.runnables import RunnableLambda
from langgraph.graph import StateGraph, END

# LCEL 封装单个 Agent 的能力
planner_runnable = RunnableLambda(lambda x: llm.invoke(f"规划：{x}").content)
coder_runnable = RunnableLambda(lambda x: llm.invoke(f"编码：{x}").content)

# LangGraph 定义协作流程
def planner_node(state):
    return {"plan": planner_runnable.invoke(state["requirement"])}

def coder_node(state):
    return {"code": coder_runnable.invoke(state["plan"])}

workflow = StateGraph(AgentState)
workflow.add_node("planner", planner_node)
workflow.add_node("coder", coder_node)
workflow.set_entry_point("planner")
workflow.add_edge("planner", "coder")
workflow.add_edge("coder", END)

graph = workflow.compile()
```

这种分层架构的优势：
- **Agent 内部**：LCEL 的函数式管道，方便单元测试和复用
- **Agent 之间**：LangGraph 的状态图，清晰表达协作逻辑

---

## 常见陷阱

| 陷阱 | 表现 | 解决方案 |
|------|------|---------|
| **LCEL 中硬编码循环** | 在 `RunnableLambda` 里写 `while`，链不可调试 | 改用 LangGraph，循环交给图管理 |
| **LangGraph 状态膨胀** | 所有历史都塞进 `state`，上下文窗口爆炸 | 只保留必要状态，历史放外部存储 |
| **节点职责不清** | 一个节点既做规划又做编码 | 遵循单一职责，一个节点一个角色 |
| **忽略检查点配置** | 生产环境用 `MemorySaver`，重启后状态丢失 | 生产用 `SqliteSaver` 或 `RedisSaver` |
| **过度工程** | 两个步骤的简单任务也上 LangGraph | 先用 LCEL，遇到循环/条件再迁移 |

---

## 参考

- [LangChain LCEL 官方文档](https://python.langchain.com/docs/concepts/lcel/)
- [LangGraph 概念指南](https://langchain-ai.github.io/langgraph/concepts/)
- [LangGraph 状态持久化](https://langchain-ai.github.io/langgraph/concepts/persistence/)
- [Human-in-the-loop 模式](https://langchain-ai.github.io/langgraph/concepts/human_in_the_loop/)
- [Multi-Agent 架构对比](research/multi-agent-architecture.md)
