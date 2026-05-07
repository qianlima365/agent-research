---
title: LangChain
date: 2026-05-07
description: 目前最流行的 LLM 应用开发框架，以 LCEL 表达式语言和模块化组件设计著称，生态覆盖从简单链路到复杂多智能体系统的全场景。
tags: [framework, python, typescript, lcel, langgraph]
---

# LangChain

**官网**: [langchain.com](https://www.langchain.com) | **GitHub**: [langchain-ai/langchain](https://github.com/langchain-ai/langchain)

> 最成熟的 LLM 应用开发框架，Python + TypeScript 双语言支持，生态覆盖从原型到生产的全链路。

---

## 概述

LangChain 是一个用于开发由语言模型驱动的应用程序的框架。它最早于 2022 年 10 月开源，迅速成长为 LLM 应用开发领域的事实标准。其核心设计哲学是**模块化**——将 LLM 应用的各个组成部分（模型调用、提示模板、工具集成、记忆管理）拆分为独立的、可组合的单元。

2024 年以后，LangChain 的重心逐渐从"Chain"（链式调用）转向 **LCEL**（LangChain Expression Language，表达式语言）和 **LangGraph**（图状态机），前者解决组件组合的优雅性问题，后者解决复杂工作流的编排问题。

---

## 核心架构

### 1. LCEL（LangChain Expression Language）

LCEL 是 LangChain 的声明式组合语法，核心运算符是管道符 `|`：

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser

# LCEL 管道：Prompt → Model → Parser
chain = (
    ChatPromptTemplate.from_template("讲一个关于{topic}的笑话")
    | ChatOpenAI(model="gpt-4")
    | StrOutputParser()
)

result = chain.invoke({"topic": "程序员"})
```

**LCEL 的设计目标**：
- **可组合**：任何实现了 `Runnable` 接口的组件都可以被 `|` 连接
- **可观察**：自动支持流式输出（`chain.stream()`）、批处理（`chain.batch()`）、异步（`chain.ainvoke()`）
- **可调试**：每个步骤的中间结果可以通过 `chain.with_config(callbacks=...)` 追踪

**并行执行**：`RunnableParallel` 让多个分支同时运行：

```python
from langchain_core.runnables import RunnableParallel

parallel = RunnableParallel(
    joke=joke_chain,
    poem=poem_chain,
    summary=summary_chain
)

# 三个链同时执行，结果合并为 dict
result = parallel.invoke({"topic": "AI"})
# {"joke": "...", "poem": "...", "summary": "..."}
```

### 2. 核心组件生态

| 组件 | 作用 | 代表类 |
|------|------|--------|
| **Model I/O** | 模型调用、输出解析 | `ChatOpenAI`, `Ollama`, `StrOutputParser` |
| **Prompts** | 提示模板管理 | `ChatPromptTemplate`, `FewShotPromptTemplate` |
| **Indexes** | 文档加载、分割、向量化 | `RecursiveCharacterTextSplitter`, `VectorStore` |
| **Retrieval** | RAG 检索增强 | `Retriever`, `MultiQueryRetriever` |
| **Tools** | 外部工具封装 | `@tool` 装饰器, `StructuredTool` |
| **Memory** | 对话历史管理 | `ConversationBufferMemory`, `VectorStoreRetrieverMemory` |
| **Agents** | 自主决策执行 | `create_react_agent`, `create_tool_calling_agent` |

### 3. LangGraph（状态机编排）

LangGraph 是 LangChain 生态的工作流编排层，基于**状态图（StateGraph）**模型：

```
          ┌─────────────┐
          │   入口节点   │
          └──────┬──────┘
                 │
        ┌────────┴────────┐
        ↓                 ↓
┌──────────────┐  ┌──────────────┐
│   节点 A     │  │   节点 B     │
│ (Agent 执行) │  │ (Agent 执行) │
└──────┬───────┘  └──────┬───────┘
       │                 │
       └────────┬────────┘
                ↓
         ┌─────────────┐
         │  条件判断    │
         │  继续/结束   │
         └──────┬──────┘
                │
         ┌──────┴──────┐
         ↓             ↓
    ┌────────┐   ┌────────┐
    │  循环   │   │  结束   │
    └────────┘   └────────┘
```

LangGraph 解决的是 LCEL 无法优雅处理的问题：**循环、条件分支、状态持久化、人机协作断点**。

**与 LCEL 的关系**：
- LCEL 负责**组件内部**的逻辑（一个 Agent 怎么调用模型、怎么解析输出）
- LangGraph 负责**组件之间**的编排（多个 Agent 怎么协作、怎么流转状态）

---

## Agent 实现方式

LangChain 提供了多种 Agent 创建模式：

### ReAct Agent（推理+行动）

```python
from langchain.agents import create_react_agent, AgentExecutor
from langchain_core.tools import tool

@tool
def search(query: str) -> str:
    """搜索网络信息"""
    return f"搜索结果：{query}"

@tool
def calculator(expression: str) -> str:
    """计算数学表达式"""
    return str(eval(expression))

tools = [search, calculator]

agent = create_react_agent(
    llm=ChatOpenAI(model="gpt-4"),
    tools=tools,
    prompt=prompt  # 包含 ReAct 格式的 system prompt
)

executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
executor.invoke({"input": "2024年GDP最高的国家是哪个？它的面积是多少万平方公里？"})
```

### Tool Calling Agent（函数调用）

针对支持原生 function calling 的模型（GPT-4、Claude、Gemini）：

```python
from langchain.agents import create_tool_calling_agent

agent = create_tool_calling_agent(
    llm=ChatOpenAI(model="gpt-4"),
    tools=tools,
    prompt=prompt
)
```

这种方式比 ReAct 更可靠，因为模型直接输出结构化函数调用，而不是让模型自己写 `"Action: search\nAction Input: ..."` 这样的文本。

---

## 记忆（Memory）机制

LangChain 的记忆系统经历了两次重大 redesign：

### v1: 基于 Memory 类的显式管理

```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory()
memory.save_context({"input": "你好"}, {"output": "你好！有什么可以帮你？"})
memory.load_memory_variables({})  # 返回完整对话历史
```

### v2: 基于消息历史的函数式管理（推荐）

```python
from langchain_core.messages import HumanMessage, AIMessage
from langchain_core.chat_history import InMemoryChatMessageHistory

# 直接用消息列表管理历史
history = InMemoryChatMessageHistory()
history.add_message(HumanMessage(content="你好"))
history.add_message(AIMessage(content="你好！"))

# 在 LCEL 链中注入历史
chain = (
    RunnablePassthrough.assign(
        history=lambda x: history.messages
    )
    | prompt
    | model
)
```

**v2 的优势**：不再依赖隐式的 `memory` 对象，状态管理更透明，更适合生产环境。

---

## RAG（检索增强生成）

LangChain 是 RAG 应用最流行的开发框架。标准流程：

```python
from langchain_community.document_loaders import WebBaseLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import Chroma

# 1. 加载文档
loader = WebBaseLoader("https://docs.langchain.com")
docs = loader.load()

# 2. 分割
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
chunks = splitter.split_documents(docs)

# 3. 向量化存储
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(chunks, embeddings)

# 4. 检索 + 生成
retriever = vectorstore.as_retriever(search_kwargs={"k": 4})

rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | ChatPromptTemplate.from_template("""
基于以下上下文回答问题：
{context}

问题：{question}
""")
    | ChatOpenAI(model="gpt-4")
    | StrOutputParser()
)

rag_chain.invoke("LangChain 的 LCEL 是什么？")
```

---

## 可观测性

LangChain 内置了完整的追踪系统：

```python
from langchain.callbacks.tracers.langchain import wait_for_all_tracers
from langsmith import Client

# 自动记录每次 invoke 的完整调用链（输入、输出、耗时、token 用量）
# 在 LangSmith 平台上可视化查看

client = Client()
# 查看 trace、评估输出质量、对比不同 prompt 的效果
```

**LangSmith**（LangChain 官方可观测性平台）支持：
- 执行追踪（Trace）：每个步骤的输入输出可视化
- 数据集与评估：批量测试、A/B 对比
- 在线调试：修改 prompt 后即时重跑

---

## 优势与局限

### 优势

| 优势 | 说明 |
|------|------|
| **生态最完善** | 支持 100+ 模型、50+ 向量数据库、30+ 文档加载器 |
| **双语言支持** | Python（功能最全）+ TypeScript（前端友好） |
| **LCEL 优雅** | 管道语法简洁，组件复用性高 |
| **LangGraph 强大** | 复杂工作流的原生支持 |
| **社区活跃** | GitHub 90k+ stars，文档完善，问题响应快 |

### 局限

| 局限 | 说明 |
|------|------|
| **学习曲线陡峭** | 概念多（Chain、Agent、Tool、Memory、Retriever），版本迭代快 |
| **过度抽象** | 简单任务引入过多封装，debug 时需要深入多层源码 |
| **版本兼容性** | 0.x 到 1.x 的迁移成本较高（Memory、Agent 等 API 重设计） |
| **性能开销** | 丰富的抽象层带来一定性能损耗，极端性能场景需手写 |

---

## 适用场景

| 场景 | 推荐用法 |
|------|---------|
| **快速原型** | LCEL 管道，几行代码搭出 MVP |
| **RAG 应用** | `load → split → embed → retrieve → generate` 标准流程 |
| **单 Agent 工具调用** | `create_tool_calling_agent` + `AgentExecutor` |
| **多 Agent 协作** | LangGraph `StateGraph` + `add_conditional_edges` |
| **企业级部署** | LangServe（REST API）+ LangSmith（可观测性）+ 自托管 Vector DB |

---

## 与其他框架的关系

```
LangChain（通用框架）
    ├── LangGraph（工作流编排）
    ├── LangSmith（可观测性）
    ├── LangServe（API 部署）
    └── 第三方集成（100+ 模型、50+ 向量库）

对比：
- 比 LlamaIndex 更通用（LlamaIndex 专注 RAG）
- 比 CrewAI 更底层（CrewAI 是高层封装，底层依赖 LangChain）
- 比 AutoGen 更工程化（AutoGen 偏向研究/对话式 Agent）
```

---

## 版本与安装

```bash
# Python
pip install langchain langchain-openai langchain-community

# TypeScript
npm install langchain @langchain/openai

# 仅 LangGraph
pip install langgraph
```

---

## 参考

- [LangChain 官方文档](https://python.langchain.com/docs/introduction/)
- [LangGraph 文档](https://langchain-ai.github.io/langgraph/)
- [LangSmith 平台](https://smith.langchain.com/)
- [LCEL 概念指南](https://python.langchain.com/docs/concepts/lcel/)
- [LangChain 与 LangGraph 多智能体实现](development/langchain-vs-langgraph-multi-agent.md)
