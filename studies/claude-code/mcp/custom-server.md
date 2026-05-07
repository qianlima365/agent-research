---
title: 自定义 MCP Server 开发
permalink: /studies/claude-code/mcp/custom-server/
---

# 自定义 MCP Server 开发

## 最小实现

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

## 配置接入

在 Claude Code 中配置自定义 MCP Server：

```json
// ~/.claude/mcp.json
{
  "mcpServers": {
    "my-server": {
      "command": "python",
      "args": ["/path/to/my_server.py"],
      "env": {"API_KEY": "..."}
    }
  }
}
```

---

## 安全最佳实践

1. **最小权限**：只暴露必要的操作
2. **输入校验**：严格校验所有参数
3. **错误处理**：不暴露内部实现细节
4. **日志记录**：记录所有调用用于审计
