---
name: rigorous-systematic-review
description: Comprehensive QA and code audits via Socratic method and pre-mortem failure simulations.
category: engineering
tags: [qa, audit, testing, socratic]
---
# Rigorous Systematic Review

Use this skill before committing major changes, before declaring a feature complete, or when reviewing critical logic.

## Review Protocol
1. **Pre-Mortem Simulation:** Assume the codebase has crashed in production immediately after your change. List the top 3 most likely reasons why it failed, and check those specific files now.
2. **Socratic Code Audit:** Ask yourself hard questions: "Is this class thread-safe?", "Does this call block the main UI thread?", "What happens if the network drops mid-request?".
3. **Refuse Performative Agreement:** Do not implement user suggestions blindly. If a requested change introduces security or performance regressions, present the trade-off clearly and propose a safer alternative.
