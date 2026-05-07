---
title: 多平台 Gateway 设计
permalink: /studies/openclaw/architecture/gateway/
---

# 多平台 Gateway 设计

Gateway 是 OpenClaw 与外部世界交互的统一入口，负责将 20+ 通信平台的消息转换为 Agent 可理解的标准格式。

---

## 核心职责

| 职责 | 说明 |
|------|------|
| **入站处理** | 接收外部平台消息，标准化为内部 Message 对象 |
| **出站处理** | 将 Agent 回复转换为各平台特有的消息格式 |
| **平台适配** | 封装各平台差异（Slack 线程、Discord Embed、微信@消息） |
| **路由分发** | 根据消息内容将请求路由到目标 Agent |

---

## Adapter 模式

```python
class Gateway:
    def __init__(self):
        self.adapters = {
            "discord": DiscordAdapter(),
            "slack": SlackAdapter(),
            "telegram": TelegramAdapter(),
            "wechat": WeChatAdapter(),
            # ... 20+ 平台
        }
```

每个 Adapter 实现统一的接口：
- `normalize(raw)` — 将平台原生格式转为标准 Message
- `denormalize(message)` — 将标准 Message 转为平台原生格式
- `send(raw)` — 发送到平台

---

## 入站流程

```
外部消息 → normalize → 路由决策 → MessageBus → 目标 Agent
```

**路由策略**：
- `@bot 查询天气` → 路由到通用助手 Agent
- `@dev-team 审查这段代码` → 路由到团队对应的 Code Reviewer Agent
- 私聊消息 → 路由到用户的个人助手 Agent

---

## 选型对比

| 维度 | OpenClaw 网关 | 自建 WebSocket | 各平台原生 SDK |
|------|--------------|---------------|--------------|
| 接入成本 | 低（统一接口） | 中 | 高（需适配每个平台） |
| 功能完整度 | 中（通用抽象） | 高（完全控制） | 高（原生能力） |
| 维护成本 | 低（社区维护 Adapter） | 高 | 高（需跟进各平台更新） |
