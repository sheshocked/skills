---
name: llm-agent-design
description: 
category: ai-tools
tags: [llm-agent-design]
---

## When to Use
Build LLM agents with tool use, planning loops, memory systems, guardrails, and multi-agent coordination.

## Agent Architecture
```
User Input → Planner (LLM) → Tool Selection → Tool Execution
                        ↑                           ↓
                        ←──── Observation ←──────────┘
                              ↓
                         Final Answer
```

## Tool Registration
```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "search_web",
            "description": "Search the web for information",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "Search query"}
                },
                "required": ["query"]
            }
        }
    }
]
```

## Planning Loop
```python
def agent_loop(user_input, max_steps=10):
    messages = [{"role": "system", "content": SYSTEM_PROMPT}, {"role": "user", "content": user_input}]

    for step in range(max_steps):
        response = llm.chat(messages, tools=tools)

        if response.tool_calls:
            for call in response.tool_calls:
                result = execute_tool(call.function.name, call.function.arguments)
                messages.append({"role": "tool", "tool_call_id": call.id, "content": result})
        else:
            return response.content  # Final answer

    return "Max steps reached"
```

## Pitfalls
- **Infinite loops**: Always set max_steps and timeout
- **Tool hallucination**: LLM may call non-existent tools — validate
- **Context overflow**: Summarize old tool results to save tokens
- **Error handling**: Retry failed tool calls with backoff

## Verification
- Test with simple single-tool tasks
- Verify multi-step reasoning works
- Check error recovery
- Measure token usage per task