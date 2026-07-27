---
name: deep-reasoning-first-principles
description: - Complex problems where surface-level analysis fails
category: engineering
tags: [deep-reasoning-first-principles]
---

## When to Use

- Complex problems where surface-level analysis fails
- Debugging issues with multiple interacting causes
- Designing solutions for requirements that seem contradictory
- Evaluating trade-offs between competing architectural approaches

## Core Concepts

- **First Principles Thinking**: Break problems into fundamental truths, reason up from there. Don't reason by analogy ("Company X does Y, so we should too").
- **Reductionism**: Decompose complex systems into simpler parts. Understand each part, then how they compose.
- **Thought Experiments**: "What if we removed X entirely?" "What if traffic was 1000x?" Extreme cases reveal assumptions.
- **Second-Order Effects**: "If we do X, then Y happens, which causes Z." Most engineering failures are second-order effects.
- **Inversion**: Instead of asking "how do I succeed?", ask "what would guarantee failure?" Then avoid those things.

## Workflow

1. **Define the actual problem** — not the symptom. "Users are slow" → "Query X takes 5 seconds because..."
2. **List all assumptions** — what must be true for your current understanding to hold?
3. **Challenge each assumption** — "What if this isn't true?" Often reveals hidden constraints.
4. **Decompose** — break the problem into independent sub-problems
5. **Solve bottom-up** — solve the hardest sub-problem first (it constrains everything else)
6. **Verify composition** — do the solutions to sub-problems compose correctly?

## Key Patterns

```python
# Structured problem decomposition
from dataclasses import dataclass, field
from typing import List, Optional

@dataclass
class ProblemAnalysis:
    symptom: str
    root_causes: List[str] = field(default_factory=list)
    assumptions: List[str] = field(default_factory=list)
    constraints: List[str] = field(default_factory=list)
    sub_problems: List["ProblemAnalysis"] = field(default_factory=list)
    solutions: List[str] = field(default_factory=list)
    second_order_effects: List[str] = field(default_factory=list)

def analyze_problem(symptom: str) -> ProblemAnalysis: