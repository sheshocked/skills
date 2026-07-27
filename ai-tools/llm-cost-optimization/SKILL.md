---
name: llm-cost-optimization
description: 
category: ai-tools
tags: [llm-cost-optimization]
---

## When to Use
Optimize LLM costs and latency: token budgeting, caching, model routing, batch APIs.

## Strategies
1. **Model routing**: Use small model for simple tasks, large for complex
2. **Caching**: Cache identical queries
3. **Batch API**: Process non-urgent requests in batch
4. **Streaming**: Improve perceived latency
5. **Token budgeting**: Limit input/output tokens

## Model Routing
```python
def route_to_model(query):
    if is_simple_query(query):
        return "gpt-4o-mini"  # Cheap
    elif is_complex_reasoning(query):
        return "gpt-4o"  # Expensive
    else:
        return "gpt-4o-mini"
```

## Pitfalls
- **Cache invalidation**: Stale cached responses
- **Routing accuracy**: Wrong model selection wastes money
- **Batch latency**: Batch APIs have higher latency

## Verification
- Track cost per query type
- Measure latency improvement
- Verify quality maintained