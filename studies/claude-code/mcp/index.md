---
title: Claude Code MCP 协议
description: Model Context Protocol 详解、生态现状和安全模型。
tags: [claude-code, mcp, protocol, tools]
---

# Claude Code MCP 协议

## 协议模型

```
┌─────────────┐         ┌─────────────┐
│  MCP Client │ ←────→ │  MCP Server │
│ (Claude Code)│  JSON-RPC │  (外部工具)  │
└─────────────┘         └─────────────┘
```

**通信方式**：JSON-RPC 2.0，标准化了工具发现、调用和结果返回。

---

## 协议流程

### 1. 工具发现（Discovery）

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

### 2. 工具调用（Invocation）

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
```

---

## 生态现状

| 类别 | 代表 Server |
|------|------------|
| **数据库** | PostgreSQL、SQLite、MySQL |
| **协作** | Slack、Notion、GitHub |
| **搜索** | Brave Search、Exa |
| **自动化** | Zapier、Make |

---

## 安全模型

MCP 的安全由两层保障：

1. **协议层**：Server 只能暴露只读或受限的操作，不能暴露危险操作
2. **应用层**：Claude Code 在调用前进行权限检查，危险操作需人类确认

```python
def call_tool(tool_name, arguments):
    if tool_name in DANGEROUS_TOOLS:
        if not human_confirm(f"确认执行 {tool_name}？"):
            raise PermissionDenied()
    return mcp_client.call(tool_name, arguments)
```

---

## 自定义 MCP Server

开发自定义 Server 只需实现三个接口：

```python
from mcp.server import Server

app = Server("my-server")

@app.list_tools()
async def list_tools():
    return [{
        "name": "my_tool",
        "description": "...",
        "input_schema": {...}
    }]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "my_tool":
        return await do_something(arguments)
```

---

## Related Notes

{% assign current_dir = page.path | split: '/' | pop | join: '/' %}
{% for doc in site.studies %}
  {% assign doc_dir = doc.path | split: '/' | pop | join: '/' %}
  {% if doc_dir == current_dir and doc.name != 'index.md' %}
- [{{ doc.title }}]({{ doc.url | relative_url }})
  {% endif %}
{% endfor %}
