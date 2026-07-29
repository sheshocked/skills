---
name: mcp-server-development
description: Build Model Context Protocol (MCP) servers in Node/TypeScript, implementing custom tools and resources.
category: ai-tools
tags: [mcp, model-context-protocol, nodejs, typescript, sdk, tools]
---

# MCP Server Development Masterclass

## When to Use
Use when extending LLM capabilities (like cursor, claude-desktop, or hermes) to query custom databases, execute shell commands, or integrate proprietary internal APIs.

## Prerequisites
- Node.js v18+.
- TypeScript configured.
- Installed `@modelcontextprotocol/sdk` package.

## Workflow
1. Initialize the MCP Server instance using STDIO transport.
2. Register tool definitions specifying parameters schema (JSON schema).
3. Implement request handler callback executing backend logic.
4. Route logs and handle process termination.

## Key Patterns

### TypeScript MCP Server (server.ts)
```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server(
  { name: "my-custom-mcp", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

// 1. Declare available tools
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "query_db",
        description: "Executes read-only SQL queries on the local SQLite state DB.",
        inputSchema: {
          type: "object",
          properties: {
            sql: { type: "string", description: "SQL query statement" }
          },
          required: ["sql"]
        }
      }
    ]
  };
});

// 2. Handle tool calls
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "query_db") {
    const sql = request.params.arguments?.sql as string;
    try {
      const results = executeQuery(sql); // local execution function
      return {
        content: [{ type: "text", text: JSON.stringify(results) }]
      };
    } catch (e: any) {
      return {
        isError: true,
        content: [{ type: "text", text: `SQL Error: ${e.message}` }]
      };
    }
  }
  throw new Error("Tool not found");
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("MCP Server running on stdio");
}

main().catch(console.error);
```

## Pitfalls
- **Writing to stdout:** MCP stdio transport uses `process.stdout` for protocol communication. Never use `console.log()` for debugging inside server scripts; redirect all logging to `console.error()`.
- **Hanging inputs:** Ensure database connections specify timeout thresholds to prevent transport blocks.

## Verification
- Test connection: Run `node build/server.js` and verify it stays open waiting for stdin.
- Test queries: Inject JSON payload `{"jsonrpc":"2.0","method":"tools/list","id":1}` via stdin and verify JSON-RPC tool lists response.
