---
name: fable-reasoning-calibration
description: Guide Fable 5's Mythos-class multi-step reasoning chains for complex tasks.
category: ai-tools
tags: [fable-5, reasoning, calibration, prompt-engineering]
---
# Fable 5 Reasoning Calibration

Use this skill when deploying Fable 5 for logic-heavy operations, long code refactors, or advanced academic analysis.

## Calibrating Fable 5
1. **Explicit Reasoning Tokens:** Allow Fable 5 to emit a "Thinking Process" block inside `<thought>` tags before outputting any code or decisions.
2. **Back-Propagation Triage:** If a step in the reasoning chain yields a contradiction, instruct the model to explicitly state "Hypothesis refuted" and reverse-trace to the branch point.
3. **Calibrated Confidence Scoring:** Ask Fable 5 to output a confidence level (0.0 to 1.0) on critical decisions, explaining its uncertainty.
