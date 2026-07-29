---
name: prompt-cache-tuning
description: Structure prompts for LLMs (such as Claude 3.5/3.7) to maximize prompt caching, reducing cost and latency.
category: ai-tools
tags: [prompt-caching, anthropic-cache, tokens-saving, prompt-engineering, cost-reduction]
---

# Prompt Cache Tuning

## When to Use
Use when querying LLMs with large context pools (e.g. code repositories, files dumps, system prompts) to trigger Claude's prompt caching. Caching saves up to 90% of input token costs and cuts latency by 2x-4x.

## Prerequisites
- Supported API (Anthropic Claude 3.5 Sonnet / 3.7 Sonnet).

## Workflow
1. Place static context (documentation, repository code, system prompts) at the *start* of the request.
2. Put dynamic elements (the user's current query) at the *end* of the request.
3. Configure cache breakpoints using the API-specific parameter block.

## Key Patterns

### Python Octokit/Requests Caching Call
```python
import os
import requests

ANTHROPIC_KEY = os.environ.get("ANTHROPIC_API_KEY")

headers = {
    "x-api-key": ANTHROPIC_KEY,
    "anthropic-version": "2024-06-01",
    "content-type": "application/json"
}

# Payload with cache breakpoints
payload = {
    "model": "claude-3-7-sonnet-20250219",
    "max_tokens": 1024,
    "system": [
        {
            "type": "text",
            "text": "You are SurfShield assistant. Here is the full repository code base...",
            # Set cache breakpoint on the system prompt block
            "cache_control": {"type": "ephemeral"}
        }
    ],
    "messages": [
        {
            "role": "user",
            "content": "How do I implement custom DNS routing?"
        }
    ]
}

r = requests.post("https://api.anthropic.com/v1/messages", json=payload, headers=headers)
response_data = r.json()

# Verify cache statistics
usage = response_data.get("usage", {})
print(f"Input tokens: {usage.get('input_tokens')}")
print(f"Cached tokens read: {usage.get('cache_read_input_tokens')}")
print(f"Cached tokens created: {usage.get('cache_creation_input_tokens')}")
```

## Pitfalls
- **Dynamic prefixes:** If you modify even a single character at the start of your prompt (e.g., prepending a timestamp), the entire cache invalidates. Keep the prefix 100% static.
- **Below minimum size:** Caching is only triggered if prompt size exceeds 1024 tokens. Don't add cache controls to short prompts.

## Verification
- Inspect response usage headers: confirm `cache_read_input_tokens` is greater than 0 on the second call.

