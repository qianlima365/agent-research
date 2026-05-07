---
title: MCP 协议流程详解
permalink: /studies/claude-code/mcp/protocol-flow/
---

# MCP 协议流程详解

## 通信方式

JSON-RPC 2.0，标准化了工具发现、调用和结果返回。

---

## 1. 工具发现（Discovery）

```json
// Client -> Server
{
  "jsonrpc": "2.0",
  "method": "tools/list",
  "id": 1
}

// Server -> Client
{
  "jsonrpc": "2.0",
  "result": {
    "tools": [
      {
        "name": "read_file",
        "description": "读取文件内容",
        "input_schema": {
          "type": "object",
          "properties": {
            "path": {"type": "string"}
          }
        }
      }
    ]
  },
  "id": 1
}
```

---

## 2. 工具调用（Invocation）

```json
// Client -> Server
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "read_file",
    "arguments": {"path": "/etc/hosts"}
  },
  "id": 2
}

// Server -> Client
{
  "jsonrpc": "2.0",
  "result": {
    "content": "127.0.0.1 localhost\n..."
  },
  "id": 2
}
```

---

## 3. 生命周期

```
连接建立 → 工具发现 → 多次调用 → 连接关闭
```

**连接复用**：一个 MCP 连接可以多次调用工具，不需要每次重新发现。
