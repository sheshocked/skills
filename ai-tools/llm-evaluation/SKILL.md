---
name: llm-evaluation
description: 
category: ai-tools
tags: [llm-evaluation]
---

## When to Use
Evaluate LLM outputs: eval harnesses, LLM-as-judge, regression testing, hallucination detection.

## LLM-as-Judge Pattern
```python
judge_prompt = (
    "Rate this response on a scale of 1-5 for:\\n"
    "- Accuracy: Is the information correct?\\n"
    "- Completeness: Does it address all parts?\\n"
    "- Clarity: Is it well-organized?\\n\\n"
    "Question: {question}\\n"
    "Response: {response}\\n\\n"
    "Rate each criterion (1-5) and explain why."
)
```

## Eval Categories
1. **Task completion**: Does it do what was asked?
2. **Factual accuracy**: Are claims verifiable?
3. **Format compliance**: Does it match expected output format?
4. **Safety**: Does it avoid harmful content?
5. **Latency**: Response time within budget?

## Pitfalls
- **Judge bias**: LLM judges may favor longer responses
- **Eval set size**: Need 50+ examples for statistical significance
- **Regression**: Track eval scores over model versions
- **Hallucination**: Hard to detect automatically — combine with fact-checking

## Verification
- Run eval suite on every model change
- Compare against baseline scores
- Test with adversarial inputs
- Check inter-annotator agreement