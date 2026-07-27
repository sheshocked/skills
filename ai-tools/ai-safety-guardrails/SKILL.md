---
name: ai-safety-guardrails
description: 
category: ai-tools
tags: [ai-safety-guardrails]
---

## When to Use
Implement AI safety: input/output filtering, PII detection, prompt injection defense, content moderation.

## Guardrail Stack
```
User Input → Input Filter → LLM → Output Filter → User
              ↓                         ↓
         PII Detection            Content Policy
         Injection Check          Toxicity Filter
```

## Input Validation
```python
import re

def check_injection(prompt):
    suspicious_patterns = [
        r"ignore previous instructions",
        r"you are now",
        r"system prompt",
        r"\\bDAN\\b",
    ]
    for pattern in suspicious_patterns:
        if re.search(pattern, prompt, re.IGNORECASE):
            return True
    return False
```

## Pitfalls
- **Over-blocking**: Filters may block legitimate content
- **Bypass**: Simple patterns can be evaded
- **Performance**: Filtering adds latency
- **False positives**: PII detection may flag non-PII

## Verification
- Test with known attack vectors
- Measure false positive/negative rates
- Verify latency impact
- Check content policy compliance