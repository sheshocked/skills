---
name: fable-economic-prompt-optimizations
description: Prompt formatting and caching strategies to minimize API costs on Fable 5 Mythos-class runs.
category: ai-tools
tags: [fable-5, prompt-caching, token-reduction, optimization]
---
# Fable 5 Economic Prompt Optimizations

Use this skill when configuring cron jobs, large codebase sweeps, or long sessions running on Fable 5.

## Optimization Rules
1. **Prompt Cache Anchors:** Format system prompts and baseline references (like locations.json schema) to maximize Fable 5's prompt caching mechanism.
2. **Context Pruning:** Automatically strip tool outputs once they are summarized. Do not carry raw, heavy logs across multiple conversation turns.
3. **Targeted Subagent Delegation:** Delegate mechanical code edits to smaller, cheaper models, keeping Fable 5 focused only on high-level reasoning and planning.
