---
name: llm-prompt-engineering
description: 
category: ai-tools
tags: [llm-prompt-engineering]
---

## When to Use
Design effective prompts for LLMs: system prompts, few-shot examples, structured output, chain-of-thought, jailbreak-resistant patterns.

## Core Concepts
- **System prompt**: Sets behavior, persona, constraints
- **Few-shot**: Examples that guide output format
- **Chain-of-thought**: Step-by-step reasoning instructions
- **Structured output**: JSON/XML schema enforcement
- **Temperature**: Controls randomness (0=deterministic, 1=creative)

## Key Patterns
```markdown
# System Prompt Template
You are a {role}. Your task is to {objective}.

## Rules
1. {rule_1}
2. {rule_2}
3. Never {forbidden_action}

## Output Format
Respond in JSON with keys: {key_1}, {key_2}

## Examples
Input: {example_input}
Output: {example_output}
```

## Structured Output (JSON mode)
```
system: "Always respond with valid JSON matching this schema: {\"action\": string, \"confidence\": number}"
user: "Classify this text: {text}"
```

## Chain-of-Thought
```
Think step by step:
1. Identify the key components
2. Analyze relationships
3. Draw conclusion
4. Verify against constraints
```

## Pitfalls
- **Overly long prompts**: Truncate context window; keep under 2000 tokens
- **Ambiguous instructions**: Be explicit about format, length, tone
- **No examples**: Few-shot dramatically improves consistency
- **Temperature too high**: For factual tasks, use 0-0.3
- **Prompt injection**: Never include untrusted user text in system prompts without framing

## Verification
- Test with 10+ varied inputs
- Check output format consistency
- Verify edge cases (empty input, adversarial input)
- Measure task completion rate