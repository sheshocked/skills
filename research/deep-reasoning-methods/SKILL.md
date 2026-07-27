---
name: deep-reasoning-methods
description: 
category: research
tags: [deep-reasoning-methods]
---

## When to Use
Apply structured reasoning: decision matrices, premortems, inversion, red-teaming.

## Decision Matrix
```markdown
| Criteria | Weight | Option A | Option B | Option C |
|---|---|---|---|---|
| Cost | 30% | 8 | 5 | 9 |
| Performance | 40% | 6 | 9 | 7 |
| Maintainability | 30% | 7 | 6 | 8 |
| Weighted Score | | 6.9 | 6.7 | 8.0 |
```

## Premortem
```markdown
## It's 6 months later and the project failed. Why?

1. Team underestimated complexity of X
2. External dependency Y was unreliable
3. Requirements changed mid-project
4. Key team member left
5. Budget was cut by 50%

## Prevention
For each failure mode → specific mitigation
```

## Inversion
```
Instead of "How do I succeed?"
Ask: "How do I guarantee failure?"

Then avoid those things.

Common failure modes:
- Try to please everyone
- Skip testing
- Ignore user feedback
- Build before validating
```

## Red Team
```markdown
## Challenge the plan:
1. What assumptions are we making?
2. What could prove us wrong?
3. Who disagrees and why?
4. What are we not seeing?
```

## Pitfalls
- **Overthinking**: Analysis paralysis — set time limits
- **Groupthink**: Seek diverse perspectives
- **Hindsight bias**: Don't let outcome affect evaluation of decision quality
- **Complexity bias**: Simple solutions are often best

## Verification
- Did you consider multiple perspectives?
- Are assumptions explicitly stated?
- Is the reasoning documented for future review?