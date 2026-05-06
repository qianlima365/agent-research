---
title: Claude Code 与 OpenClaw 多智能体架构对比
date: 2026-05-07
---

# Claude Code 与 OpenClaw 多智能体架构对比

> 两者代表了多 Agent 架构的两种哲学：一个是从 IDE 内向外的**深度集成**，一个是从通信层向内的**广泛连接**。

---

## 基本定位

| 维度 | Claude Code | OpenClaw |
|------|-------------|----------|
| **开发者** | Anthropic（官方） | 开源社区 |
| **核心场景** | 代码开发、工程任务 | 多渠道自动化、工作流编排 |
| **执行环境** | 本地终端 / IDE 内嵌 | 本地终端 + 多平台网关 |
| **模型绑定** | 深度绑定 Claude 系列 | 模型无关（OpenAI/Anthropic/Ollama/Qwen/Gemini） |
| **开源协议** | 闭源（免费使用） | MIT License |

**本质差异**：Claude Code 是**垂直工具**（为写代码而设计），OpenClaw 是**水平平台**（为连接一切而设计）。

---

## Agent 定义方式

### Claude Code：文件 + 目录约定

Agent 定义放在 `.claude/agents/`（项目级）或 `~/.claude/agents/`（用户级）：

```yaml
# ~/.claude/agents/backend-architect.md
name: backend-architect
description: 设计 RESTful API、微服务边界和数据库 schema
model: sonnet
tools: [Read, Write, Edit, Bash]

You are a senior backend architect specializing in scalable system design...
```

- **加载机制**：自动发现目录下的 `.md` 文件
- **作用域**：项目级 Agent 只在该项目可用，用户级 Agent 全局可用
- **格式**：YAML frontmatter + Markdown 主体，结构简单直接

### OpenClaw：三文件体系

每个 Agent 是一个包含三个文件的 workspace：

```
~/.openclaw/agency-agents/
├── planner/
│   ├── SOUL.md      ← 灵魂/核心人格
│   ├── AGENTS.md    ← 协作关系定义
│   └── IDENTITY.md  ← 身份信息与能力边界
├── coder/
│   ├── SOUL.md
│   ├── AGENTS.md
│   └── IDENTITY.md
```

| 文件 | 作用 |
|------|------|
| **SOUL.md** | Agent 的核心人格、价值观、决策风格 |
| **AGENTS.md** | 与其他 Agent 的协作契约（输入/输出/依赖） |
| **IDENTITY.md** | 身份标识、能力清单、不处理事项 |

- **加载机制**：通过 `agentId` 注册到 OpenClaw 会话
- **作用域**：Agent 之间通过 `agentToAgent` 工具直接调用
- **格式**：纯 Markdown，语义分层更明确

**对比**：Claude Code 的 Agent 定义是**扁平的**（一个文件搞定），OpenClaw 是**结构化的**（三文件分离人格、协作、身份）。

---

## 协作架构

### Claude Code：Orchestrator-Worker（中心辐射）

```
    ┌─────────────────────────┐
    │    主 Claude 会话        │  ← Orchestrator
    │  （人类直接交互的入口）    │
    └──────┬────────┬─────────┘
           │        │
     ┌─────┘        └─────┐
     ↓                    ↓
┌─────────┐         ┌─────────┐
│ Agent A │         │ Agent B │  ← Worker（隔离上下文）
│ 只读     │         │ 读写    │
└─────────┘         └─────────┘
```

- **通信方式**：所有消息通过主会话中转，Agent 之间**不能直接对话**
- **上下文隔离**：每个子 Agent 有独立的上下文窗口，只接收任务相关信息
- **Ultra Plan 模式**：3 个 Explorer Agent 并行探索 + 1 个 Critic Agent 评估，模拟团队头脑风暴

### OpenClaw：去中心化网状协作

```
┌─────────┐ ←────→ ┌─────────┐
│ Agent A │        │ Agent B │
└────┬────┘        └────┬────┘
     │    ↘        ↗    │
     └──────→ ┌──┐ ←─────┘
              │C │
              └─┬┘
                ↓
           ┌─────────┐
           │  Cron   │  ← 定时触发
           │ Scheduler│
           └─────────┘
```

- **通信方式**：Agent 通过 `agentToAgent` 工具**直接对话**，或通过共享文件/线程间接协作
- **上下文共享**：通过 Convex 后端或文件系统共享状态，无强制隔离
- **Cron 驱动**：Agent 按预设时间表唤醒（`clawion agent wake`），适合 7×24 小时值守场景

**对比**：Claude Code 是**强控制的中心调度**，OpenClaw 是**弱控制的自发协作**。

---

## 技术架构与伪代码实现

### Claude Code：分层隔离架构

Claude Code 的多 Agent 系统可以抽象为四层：

```
┌─────────────────────────────────────────────┐
│  Layer 4: Orchestrator（主会话调度器）         │
│  - 解析用户意图，决定调用哪个子 Agent           │
│  - 管理子 Agent 的生命周期（创建/销毁/暂停）    │
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

**伪代码：Orchestrator-Worker 模式**

```python
class Orchestrator:
    def __init__(self):
        self.agent_pool = {}          # agent_id -> SubAgent
        self.mcp_client = MCPClient() # 统一工具客户端
        self.session_context = []     # 主会话历史

    def dispatch(self, user_input: str) -> str:
        """解析用户意图，调度子 Agent"""
        # 1. 意图路由：决定需要哪些专家
        required_agents = self._route_intent(user_input)

        results = []
        for agent_id in required_agents:
            # 2. 创建或复用子 Agent（隔离上下文）
            agent = self._get_or_create_agent(agent_id)

            # 3. 构建任务包（只传递必要上下文）
            task_package = {
                "task": user_input,
                "context": self._filter_context(agent_id, self.session_context),
                "tools": agent.allowed_tools  # 权限受限的工具列表
            }

            # 4. 执行子任务
            result = agent.execute(task_package)
            results.append(result)

        # 5. 汇总结果，更新主会话
        final = self._synthesize(results)
        self.session_context.append(("user", user_input))
        self.session_context.append(("assistant", final))
        return final

    def _get_or_create_agent(self, agent_id: str) -> SubAgent:
        if agent_id not in self.agent_pool:
            config = load_agent_config(f"~/.claude/agents/{agent_id}.md")
            self.agent_pool[agent_id] = SubAgent(
                config=config,
                mcp_client=self.mcp_client,  # 共享 MCP 客户端
                read_only=True               # 默认只读，需显式授权
            )
        return self.agent_pool[agent_id]


class SubAgent:
    def __init__(self, config, mcp_client, read_only=True):
        self.config = config
        self.mcp = mcp_client
        self.read_only = read_only
        self.context_window = []  # 独立上下文，与主会话隔离

    def execute(self, task_package: dict) -> str:
        """子 Agent 执行逻辑"""
        # 加载 Agent 人格定义
        system_prompt = self.config.system_message

        # 构建消息历史（仅包含本次任务相关上下文）
        messages = [
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": task_package["task"]},
            {"role": "system", "content": f"可用工具：{task_package['tools']}"}
        ]

        # 调用 LLM，可能产生工具调用请求
        response = llm.chat(messages)

        # 如果请求工具调用，通过 MCP 客户端执行（受权限限制）
        if response.tool_calls:
            for tool_call in response.tool_calls:
                if self.read_only and tool_call.name in WRITE_TOOLS:
                    raise PermissionError(f"只读 Agent 不能调用 {tool_call.name}")
                tool_result = self.mcp.call(tool_call)
                messages.append({"role": "tool", "content": tool_result})

        # 生成最终回复
        final = llm.chat(messages)
        return final.content
```

**伪代码：Ultra Plan（三探索 + 一评审）**

```python
class UltraPlan:
    """Claude Code Ultra 的并行探索模式"""

    def solve(self, problem: str) -> str:
        # 1. 启动 3 个 Explorer Agent（并行）
        explorers = [
            SubAgent(config=load("explorer-1.md"), strategy="保守"),
            SubAgent(config=load("explorer-2.md"), strategy="激进"),
            SubAgent(config=load("explorer-3.md"), strategy="创新"),
        ]

        solutions = []
        with ThreadPoolExecutor(max_workers=3) as executor:
            futures = [
                executor.submit(exp.execute, problem)
                for exp in explorers
            ]
            for future in futures:
                solutions.append(future.result())

        # 2. 启动 Critic Agent（独立评审，避免自我偏差）
        critic = SubAgent(config=load("critic.md"), read_only=True)
        evaluation = critic.execute(
            f"请客观评审以下三个方案，选出最优：\n"
            f"方案A：{solutions[0]}\n"
            f"方案B：{solutions[1]}\n"
            f"方案C：{solutions[2]}"
        )

        # 3. 返回最优方案 + 评审理由
        best = self._extract_best(evaluation, solutions)
        return f"{best}\n\n【评审意见】{evaluation}"
```

---

### OpenClaw：注册中心 + 消息总线架构

OpenClaw 的多 Agent 系统可以抽象为三层：

```
┌─────────────────────────────────────────────┐
│  Layer 3: Gateway（多平台网关）                │
│  - WebSocket 统一接入层                        │
│  - 适配 WhatsApp/Discord/Slack/飞书 等 20+ 平台 │
│  - 消息格式标准化（入站/出站转换）              │
├─────────────────────────────────────────────┤
│  Layer 2: Agent Registry + Message Bus        │
│  - Agent 注册中心（agentId -> Agent 实例）      │
│  - agentToAgent 工具（直接 RPC 调用）           │
│  - Cron Scheduler（定时唤醒调度器）             │
├─────────────────────────────────────────────┤
│  Layer 1: Agent Workers（Agent 工作池）        │
│  - 每个 Agent 加载 SOUL.md + AGENTS.md        │
│  - 独立进程/线程运行，常驻内存                  │
│  - 通过共享文件/Convex DB 持久化状态            │
└─────────────────────────────────────────────┘
```

**伪代码：Agent 注册与发现**

```python
class AgentRegistry:
    """Agent 注册中心，管理所有活跃 Agent"""

    def __init__(self):
        self.agents: dict[str, Agent] = {}  # agentId -> Agent
        self.message_bus = MessageBus()      # 内部消息总线

    def register(self, workspace_path: str):
        """注册一个 Agent（从 workspace 目录加载）"""
        agent_id = Path(workspace_path).name

        # 加载三文件体系
        soul = load_markdown(f"{workspace_path}/SOUL.md")
        agents_contract = load_markdown(f"{workspace_path}/AGENTS.md")
        identity = load_markdown(f"{workspace_path}/IDENTITY.md")

        agent = Agent(
            agent_id=agent_id,
            soul=soul,              # 核心人格
            contract=agents_contract,  # 协作契约
            identity=identity,      # 身份边界
            llm=create_llm(identity.model_preference)  # 按偏好选择模型
        )

        self.agents[agent_id] = agent
        self.message_bus.subscribe(agent_id, agent.on_message)
        return agent

    def discover(self, capability: str) -> list[str]:
        """根据能力发现 Agent（用于动态组队）"""
        matches = []
        for agent_id, agent in self.agents.items():
            if capability in agent.identity.capabilities:
                matches.append(agent_id)
        return matches


class Agent:
    """单个 Agent 实例，常驻运行"""

    def __init__(self, agent_id, soul, contract, identity, llm):
        self.agent_id = agent_id
        self.soul = soul
        self.contract = contract      # 定义了输入/输出/依赖接口
        self.identity = identity      # 能力清单 + 不处理事项
        self.llm = llm
        self.state = {}               # 运行时状态（可持久化）
        self.memory = VectorStore()   # 长期记忆（向量检索）

    def on_message(self, message: Message):
        """接收消息并处理"""
        # 1. 判断消息类型
        if message.type == "direct":
            # 直接请求：执行任务
            result = self.execute(message.content, message.from_agent)
            self.send_reply(message.from_agent, result)

        elif message.type == "broadcast":
            # 广播消息：判断是否与自己相关
            if self._is_relevant(message.content):
                result = self.execute(message.content)
                self.broadcast(result)

        elif message.type == "cron_wake":
            # Cron 唤醒：检查待办事项
            pending = self._check_pending_tasks()
            for task in pending:
                result = self.execute(task)
                self.state[task.id] = {"status": "done", "result": result}
                self.persist_state()

    def execute(self, task: str, from_agent: str = None) -> str:
        """执行任务的核心逻辑"""
        # 1. 检查是否在能力范围内
        if not self._can_handle(task):
            return f"【{self.agent_id}】此任务超出我的能力范围：{self.identity.cannot_handle}"

        # 2. 检索相关记忆
        relevant_memories = self.memory.search(task, top_k=3)

        # 3. 构建提示（注入人格 + 契约 + 记忆）
        prompt = f"""
        {self.soul.core_values}
        {self.soul.decision_style}

        协作契约：
        {self.contract.input_contract}
        {self.contract.output_contract}

        相关历史记忆：
        {relevant_memories}

        当前任务：{task}
        请求来源：{from_agent or 'user'}
        """

        # 4. 调用 LLM
        response = self.llm.generate(prompt)

        # 5. 更新记忆
        self.memory.add(f"Task: {task}\nResult: {response}")

        return response

    def agent_to_agent(self, target_agent_id: str, task: str) -> str:
        """调用另一个 Agent（OpenClaw 的核心协作原语）"""
        if target_agent_id not in registry.agents:
            raise AgentNotFoundError(target_agent_id)

        # 检查契约：我是否有权限调用它？
        if target_agent_id not in self.contract.dependencies:
            raise ContractViolationError(f"未声明对 {target_agent_id} 的依赖")

        target = registry.agents[target_agent_id]
        return target.execute(task, from_agent=self.agent_id)
```

**伪代码：Cron 调度与常驻运行**

```python
class CronScheduler:
    """OpenClaw 的定时唤醒调度器"""

    def __init__(self, registry: AgentRegistry):
        self.registry = registry
        self.schedules: dict[str, str] = {}  # agentId -> cron_expression

    def schedule(self, agent_id: str, cron: str):
        """为 Agent 注册定时任务"""
        self.schedules[agent_id] = cron
        # 实际实现：APScheduler / Celery Beat

    def run(self):
        """主循环：持续检查是否有 Agent 需要唤醒"""
        while True:
            for agent_id, cron_expr in self.schedules.items():
                if self._should_wake(cron_expr):
                    agent = self.registry.agents[agent_id]
                    # 发送唤醒消息（非阻塞）
                    self.registry.message_bus.send(
                        to=agent_id,
                        message=Message(type="cron_wake", content="check_pending")
                    )
            sleep(60)  # 每分钟检查一次


class Gateway:
    """多平台消息网关"""

    def __init__(self):
        self.adapters = {
            "discord": DiscordAdapter(),
            "slack": SlackAdapter(),
            "telegram": TelegramAdapter(),
            "wechat": WeChatAdapter(),
            # ... 20+ 平台
        }

    def on_inbound(self, platform: str, raw_message: dict):
        """接收外部平台消息"""
        # 1. 格式标准化
        msg = self.adapters[platform].normalize(raw_message)

        # 2. 路由到目标 Agent
        target_agent = self._route(msg)

        # 3. 转发到 Agent Registry
        registry.message_bus.send(
            to=target_agent,
            message=Message(type="direct", content=msg.text, from_user=msg.user_id)
        )

    def on_outbound(self, agent_id: str, response: str, platform: str):
        """将 Agent 回复发送到外部平台"""
        raw = self.adapters[platform].denormalize(response)
        self.adapters[platform].send(raw)
```

---

## 核心实现差异对比

| 维度 | Claude Code 实现 | OpenClaw 实现 |
|------|-----------------|---------------|
| **Agent 创建** | 按需实例化（任务来才创建） | 预注册常驻（启动时加载所有 Agent） |
| **通信机制** | 通过 Orchestrator 中转（星型） | agentToAgent 直接调用（网状） |
| **上下文隔离** | 强隔离（每个子 Agent 独立窗口） | 弱隔离（共享文件/数据库） |
| **状态持久化** | 会话级（结束即丢失） | 持久化（SQLite/Convex/文件） |
| **调度方式** | 事件驱动（用户输入触发） | Cron + 事件混合（常驻值守） |
| **工具调用** | MCP Client 统一代理 | Agent 内部直接调用 Skills |
| **权限控制** | Orchestrator 集中管控 | 契约声明 + 运行时检查 |
| **错误恢复** | Orchestrator 重试/降级 | Agent 自主处理 + 状态回滚 |

**关键洞察**：Claude Code 的架构像**现代操作系统**（内核调度进程，进程隔离运行），OpenClaw 的架构像**微服务网格**（服务注册发现，服务间直接调用，统一网关接入）。

---

## 状态与记忆管理

| 维度 | Claude Code | OpenClaw |
|------|-------------|----------|
| **短期记忆** | 各 Agent 独立的上下文窗口 | 当前会话的 thread/message 历史 |
| **中期记忆** | `CLAUDE.md` 文件（项目级共享上下文） | 共享文件 / Convex 数据库 |
| **长期记忆** | 依赖外部 MCP 服务（向量数据库） | ClawHub skills + 本地 SQLite |
| **状态持久化** | 会话结束即丢失（除非写入文件） | Cron 调度持久运行，状态持续 |

**关键差异**：Claude Code 的 Agent 是**按需启动**的（任务来才唤醒），OpenClaw 的 Agent 是**常驻运行**的（像后台服务一样持续存在）。

---

## 工具集成机制

### Claude Code：MCP 深度集成

```
Claude Code ←──MCP──→ 外部工具
            │
            ├─── FileSystem（本地文件读写）
            ├─── GitHub（PR、Issue 操作）
            ├─── Database（SQL 查询）
            ├─── Browser（网页抓取）
            └─── 自定义 MCP Server
```

- **MCP（Model Context Protocol）**：Anthropic 提出的标准协议，工具以 Server 形式暴露，Claude Code 作为 Client 连接
- **生态**：社区已开发大量 MCP Server（Postgres、Slack、Notion、Zapier 等）
- **安全**：工具权限分级，危险操作需要人类确认

### OpenClaw：Skills + Gateway 扩展

```
OpenClaw ←──Skill──→ 自动化能力
         │
         ├─── WebSocket Gateway ──→ WhatsApp/Discord/Slack/Telegram
         ├─── ClawHub（50+ skills）
         └─── 自定义 Agent skills
```

- **Skills**：可复用的自动化模块，类似插件
- **Gateway**：统一的消息网关，连接 20+ 通信平台
- **生态**：ClawHub 提供现成 skills，社区持续贡献

**对比**：Claude Code 的集成是**工具导向**的（连接开发工具），OpenClaw 是**渠道导向**的（连接通信平台）。

---

## 安全与权限

| 维度 | Claude Code | OpenClaw |
|------|-------------|----------|
| **权限模型** | 工具级限制（只读/读写）+ 人类确认 | 配置文件控制 + Gateway 权限 |
| **Agent 隔离** | 强隔离（独立上下文、独立权限） | 弱隔离（共享文件、直接通信） |
| **危险操作** | 必须人类确认（删除文件、执行命令等） | 依赖配置约束 |
| **可审计性** | 操作日志完整可追溯 | 依赖外部系统（如 Convex） |

Claude Code 在安全性上明显更保守——子 Agent 默认是**只读**的，只有 Orchestrator 能决定给它什么权限。OpenClaw 更灵活，但也意味着更多配置责任落在用户身上。

---

## 适用场景对比

| 场景 | 推荐选择 | 理由 |
|------|---------|------|
| **代码审查与重构** | Claude Code | 深度 IDE 集成、代码感知、MCP 工具链 |
| **7×24 自动化值守** | OpenClaw | Cron 调度、多平台通知、常驻运行 |
| **复杂任务分解** | Claude Code | Orchestrator 模式天然适合任务拆分 |
| **跨团队协作** | OpenClaw | 直接对接 Slack/Discord/飞书等团队工具 |
| **快速原型开发** | Claude Code | 代码生成、测试、调试闭环 |
| **多模型对比实验** | OpenClaw | 模型无关，可同时跑 GPT-4、Claude、Kimi |

---

## 架构哲学总结

| Claude Code | OpenClaw |
|-------------|----------|
| **中央计划** | **自发秩序** |
| 一个聪明的 Orchestrator 分配任务给专家 | 一群平等的 Agent 自主协商 |
| **确定性优先** | **灵活性优先** |
| 严格控制流程，确保可预测性 | 容忍不确定性，追求适应性 |
| **深度集成** | **广泛连接** |
| 把少数工具用到极致 | 把多数平台连接到一起 |
| **人类在环** | **无人值守** |
| 每个关键决策有人类把关 | Agent 可以独立运行数天 |

---

## 实践建议

1. **如果你是开发者，主要写代码** → 从 Claude Code 开始，它的代码感知能力和 MCP 生态是其他工具难以替代的

2. **如果你需要 7×24 自动化** → 用 OpenClaw，Cron 调度 + 多平台网关是核心竞争力

3. **两者可以互补**：用 Claude Code 做核心开发，用 OpenClaw 做外部通知和监控，通过 MCP 或文件系统桥接

4. **无论选哪个，先定义好 AGENTS.md** — 这正在成为跨工具的标准（Claude Code、Codex CLI、OpenClaw 都支持）

---

## 参考

- [Claude Code 文档](https://code.claude.com/docs)
- [Claude Code Ultra Plan 多 Agent 架构](https://www.mindstudio.ai/blog/claude-code-ultra-plan-multi-agent-architecture/)
- [OpenClaw Agents 一键部署](https://github.com/shenhao-stu/openclaw-agents)
- [Fleet — 多 Agent 舰队管理](https://github.com/oguzhnatly/fleet)
- [Clawe — 多 Agent 协调系统](https://github.com/getclawe/clawe)
- [AGENTS.md 工具无关标准](https://github.com/affaan-m/everything-claude-code/blob/main/.codex/AGENTS.md)
