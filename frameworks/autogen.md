---
title: AutoGen
date: 2026-05-07
description: 微软推出的多智能体对话框架，核心哲学是"对话即编程"——通过多个 ConversableAgent 之间的自然语言对话协作完成任务。
tags: [framework, microsoft, multi-agent, conversation, code-execution]
---

# AutoGen

**官网**: [microsoft.github.io/autogen](https://microsoft.github.io/autogen/) | **GitHub**: [microsoft/autogen](https://github.com/microsoft/autogen)

> 微软推出的多智能体对话框架，核心哲学是"对话即编程"——让多个 AI Agent 像人类团队一样通过自然语言对话协作完成任务。

---

## 概述

AutoGen 是微软研究院于 2023 年开源的多智能体对话框架。与 LangChain 的"管道组合"或 LangGraph 的"状态机"不同，AutoGen 的核心抽象是**对话（Conversation）**——Agent 之间通过发送和接收消息来协作，人类也可以随时加入对话。

这种设计的灵感来自人类团队协作：**不是预先定义好 rigid 的工作流，而是通过对话动态协商任务分配、澄清需求、验证结果**。

---

## 核心架构

### 1. ConversableAgent — 一切的基础

AutoGen 中所有 Agent 都继承自 `ConversableAgent`，它具备三个核心能力：

| 能力 | 说明 |
|------|------|
| **发送消息** | `send(message, recipient)` 向另一个 Agent 发送消息 |
| **接收消息** | `receive(message, sender)` 收到消息后触发回复 |
| **生成回复** | `generate_reply(messages, sender)` 基于上下文生成回应 |

```python
from autogen import ConversableAgent

# 最基本的两个 Agent 对话
agent_a = ConversableAgent(
    name="planner",
    system_message="你是产品规划专家",
    llm_config={"config_list": [{"model": "gpt-4", "api_key": "..."}]}
)

agent_b = ConversableAgent(
    name="coder",
    system_message="你是开发工程师",
    llm_config={"config_list": [{"model": "gpt-4", "api_key": "..."}]}
)

# 发起对话：planner 向 coder 发送消息
agent_a.initiate_chat(
    recipient=agent_b,
    message="我们需要一个用户登录功能，请设计实现方案"
)
```

### 2. 内置 Agent 类型

| Agent 类型 | 角色 | 特点 |
|-----------|------|------|
| **AssistantAgent** | AI 助手 | 默认的 LLM 驱动 Agent，擅长推理和生成 |
| **UserProxyAgent** | 人类代理 | 代表人类用户，可以执行代码、调用工具，或转发给真实人类 |
| **GroupChatManager** | 群聊管理员 | 管理多个 Agent 的群聊，决定下一个发言者 |

**AssistantAgent + UserProxyAgent** 是 AutoGen 最经典的组合模式：

```python
from autogen import AssistantAgent, UserProxyAgent

assistant = AssistantAgent(
    name="assistant",
    system_message="你是一个 helpful AI 助手",
    llm_config={"config_list": [{"model": "gpt-4", "api_key": "..."}]}
)

# UserProxyAgent 可以执行代码
user_proxy = UserProxyAgent(
    name="user_proxy",
    human_input_mode="NEVER",  # 不询问人类，自动执行
    code_execution_config={
        "work_dir": "./coding",
        "use_docker": False      # 本地执行（生产建议用 Docker）
    }
)

# 启动对话：UserProxyAgent 提需求，AssistantAgent 写代码
user_proxy.initiate_chat(
    assistant,
    message="写一个 Python 函数计算斐波那契数列"
)
```

**执行流程**：

```
UserProxyAgent: "写一个 Python 函数计算斐波那契数列"
    ↓
AssistantAgent: "```python\ndef fib(n):...\n```"
    ↓
UserProxyAgent: （自动提取代码并执行）
    ↓
UserProxyAgent: "代码执行成功，结果是：[0, 1, 1, 2, 3, 5]"
    ↓
AssistantAgent: "验证通过，函数工作正常。"
```

### 3. 代码执行机制

AutoGen 的 `UserProxyAgent` 内置代码执行能力，这是它与其它框架最大的差异化设计：

```python
user_proxy = UserProxyAgent(
    name="user_proxy",
    code_execution_config={
        "work_dir": "./workspace",     # 代码工作目录
        "use_docker": True,             # 用 Docker 隔离执行（推荐）
        "timeout": 120,                 # 执行超时（秒）
        "last_n_messages": 3            # 从最近 3 条消息中提取代码
    }
)
```

**代码提取逻辑**：
1. AssistantAgent 在回复中用 markdown 代码块包裹代码
2. UserProxyAgent 自动识别 ```python ... ``` 格式的代码
3. 将代码写入临时文件并执行
4. 捕获 stdout/stderr 作为执行结果返回

**安全机制**：
- Docker 隔离执行（推荐生产环境使用）
- 超时控制防止死循环
- 可配置允许的命令白名单

---

## 对话模式

### 模式一：两两对话（Two-Agent Chat）

最基础的模式，两个 Agent 一对一交流：

```python
user_proxy.initiate_chat(assistant, message="帮我写一个爬虫")
```

**终止条件**：
- 达到最大轮数 (`max_turns`)
- AssistantAgent 回复包含特定终止词（如 "TERMINATE"）
- 人类手动终止

### 模式二：群聊（Group Chat）

多个 Agent 在同一个群里协作，由 `GroupChatManager` 决定发言顺序：

```python
from autogen import GroupChat, GroupChatManager

# 定义多个专家 Agent
planner = AssistantAgent(name="planner", system_message="你是规划专家...")
coder = AssistantAgent(name="coder", system_message="你是开发专家...")
tester = AssistantAgent(name="tester", system_message="你是测试专家...")

# 创建群聊
group_chat = GroupChat(
    agents=[user_proxy, planner, coder, tester],
    messages=[],
    max_round=20
)

# 群聊管理员决定下一个谁发言
manager = GroupChatManager(
    groupchat=group_chat,
    llm_config={"config_list": [{"model": "gpt-4", "api_key": "..."}]}
)

# 启动群聊
user_proxy.initiate_chat(
    manager,
    message="我们需要开发一个电商网站的购物车功能"
)
```

**发言顺序策略**：

| 策略 | 说明 |
|------|------|
| `round_robin` | 轮流发言 |
| `random` | 随机选择 |
| `manual` | 人类手动指定 |
| `auto`（默认） | LLM 根据上下文智能选择最合适的下一个发言者 |

`auto` 策略的核心 prompt：

```
根据当前对话，选择下一个应该发言的人。
可选：[user_proxy, planner, coder, tester]
考虑：谁最相关？谁有信息要补充？
```

### 模式三：嵌套对话（Nested Chat）

Agent A 在与 Agent B 对话的过程中，可以临时"拉个群"与 Agent C、D 讨论，然后再回到与 B 的对话：

```python
# 定义内部讨论用的子 Agent
reviewer = AssistantAgent(name="reviewer", system_message="你是代码审查专家")

# 配置 coder 在回复前，先与 reviewer 进行内部讨论
coder.register_nested_chats(
    trigger=lambda sender: sender == planner,  # 当 planner 发来消息时触发
    chat_queue=[
        {
            "recipient": reviewer,
            "message": "请审查以下代码方案",
            "max_turns": 2
        }
    ]
)
```

**使用场景**：
- 编码前先咨询架构师
- 生成回复前让审查 Agent 把关
- 复杂任务需要内部头脑风暴

---

## 函数调用（Tool Use）

AutoGen 支持通过 `@register_for_llm` 和 `@register_for_execution` 装饰器注册工具：

```python
from autogen import register_function

# 定义工具
def search_web(query: str) -> str:
    """搜索网络信息"""
    return f"搜索结果：{query}"

def calculate(expression: str) -> float:
    """计算数学表达式"""
    return eval(expression)

# 注册给 AssistantAgent（LLM 可以调用）
register_function(
    search_web,
    caller=assistant,
    executor=user_proxy,
    description="搜索网络信息"
)

register_function(
    calculate,
    caller=assistant,
    executor=user_proxy,
    description="计算数学表达式"
)
```

**执行流程**：

```
AssistantAgent: "我需要搜索一下最新数据"
    ↓（LLM 生成函数调用）
UserProxyAgent: （执行 search_web 函数）
    ↓（返回结果）
AssistantAgent: "根据搜索结果..."
```

与 LangChain 的 Tool 对比：
- AutoGen 的函数调用是**对话驱动的**——LLM 在对话中决定何时调用
- LangChain 的 Tool 是**管道驱动的**——在预定义的流程节点调用

---

## 与 LangChain / LangGraph 的对比

| 维度 | AutoGen | LangChain + LangGraph |
|------|---------|----------------------|
| **核心抽象** | 对话（Conversation） | 管道（LCEL）/ 状态图（StateGraph） |
| **协作方式** | 自然语言协商 | 预定义流程 |
| **代码执行** | **内置**，UserProxyAgent 自动执行 | 需自行集成（PythonREPLTool 等） |
| **人类介入** | **原生支持**，可随时加入对话 | 需用 LangGraph `interrupt` 实现 |
| **灵活性** | 高（对话驱动，动态调整） | 中（流程预定义，确定性高） |
| **可控性** | 中（对话可能偏离轨道） | 高（状态机精确控制） |
| **调试难度** | 较高（对话历史长，不易追踪） | 中（LangSmith 可视化完善） |
| **适用场景** | 探索性任务、创意协作、代码生成 | 确定性流程、企业级工作流 |

**一句话区分**：
- AutoGen = **"我们一起聊聊怎么做"**（对话驱动，灵活但可能发散）
- LangGraph = **"按这个流程一步一步做"**（流程驱动，可控但需预先设计）

---

## 优势与局限

### 优势

| 优势 | 说明 |
|------|------|
| **代码执行原生** | UserProxyAgent 自动提取并执行代码，无需额外配置 |
| **人类在环自然** | 人类作为对话参与者直接介入，不需要断点机制 |
| **动态协作** | 对话过程中可以临时调整方向、追加需求 |
| **群聊智能调度** | GroupChatManager 自动选择下一个最合适的 Agent |
| **Docker 隔离** | 代码执行默认支持 Docker，安全性有保障 |

### 局限

| 局限 | 说明 |
|------|------|
| **对话发散** | 多 Agent 对话容易偏离主题，需要精心设计 system message |
| **token 消耗高** | 每个 Agent 都有独立的上下文，群聊时 token 用量成倍增长 |
| **调试困难** | 对话历史长，出问题后难以定位是哪个 Agent 的哪个回复导致的 |
| **状态不透明** | 没有 LangGraph 那样的全局状态可视化 |
| **学习曲线** | 需要理解对话流、嵌套聊天、终止条件等概念 |

---

## 适用场景

| 场景 | 为什么适合 AutoGen |
|------|-------------------|
| **自动编程** | UserProxyAgent 自动执行代码，闭环验证 |
| **数据科学探索** | 代码 → 结果 → 分析 → 新代码的循环 |
| **创意头脑风暴** | 多个 Agent 自由讨论，碰撞想法 |
| **复杂任务分解** | Agent 之间通过对话协商分工 |
| **人机协作** | 人类随时加入对话提供反馈 |

**不适合的场景**：
- 需要严格合规审批流程的企业工作流（LangGraph 更适合）
- 对 token 成本敏感的场景（群聊 token 消耗高）
- 需要精确控制每一步的执行顺序（流程驱动更合适）

---

## 版本与安装

```bash
# Python（0.2.x 是稳定版）
pip install pyautogen

# 如果需要 Docker 代码执行
docker --version  # 确保 Docker 已安装

# 最新开发版
pip install https://github.com/microsoft/autogen/archive/main.zip
```

---

## 参考

- [AutoGen 官方文档](https://microsoft.github.io/autogen/)
- [AutoGen GitHub](https://github.com/microsoft/autogen)
- [AutoGen Paper: Enabling Next-Gen LLM Applications via Multi-Agent Conversation](https://arxiv.org/abs/2308.08155)
- [LangChain 框架研究](frameworks/langchain.md)
- [LangGraph 框架研究](frameworks/langgraph.md)
- [多智能体基础架构研究](research/multi-agent-architecture.md)
