---
name: mcp-server-development
description: 
category: ai-tools
tags: [mcp-server-development]
---

## When to Use
Build Model Context Protocol (MCP) servers: tools, resources, transports, authentication.

## MCP Architecture
```
LLM Client ↔ MCP Transport (stdio/SSE) ↔ MCP Server
                                            ↓
                                      Tools/Resources
```

## Server Implementation
```python
from mcp.server import Server
from mcp.types import Tool, TextContent

server = Server("my-tools")

@server.list_tools()
async def list_tools():
    return [
        Tool(name="search", description="Search documents", inputSchema={
            "type": "object",
            "properties": {"query": {"type": "string"}},
            "required": ["query"]
        })
    ]

@server.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "search":
        results = await search_docs(arguments["query"])
        return [TextContent(type="text", text=str(results))]
```

## Pitfalls
- **Tool descriptions**: Must be clear for LLM to choose correctly
- **Input validation**: Always validate tool inputs
- **Error handling**: Return meaningful error messages
- **Transport**: stdio for local, SSE for remote

## Verification
- Test with MCP Inspector
- Verify tool calls from LLM client
- Check error handling