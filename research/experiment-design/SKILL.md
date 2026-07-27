---
name: experiment-design
description: 
category: research
tags: [experiment-design]
---

## When to Use
Design A/B tests and experiments: hypothesis, metrics, sample size, statistical significance.

## A/B Test Template
```markdown
## Hypothesis
If we {change}, then {metric} will {improve/decrease} by {amount}.

## Design
- Control: Current version
- Treatment: New version
- Metric: {primary metric}
- Sample size: {calculated n}
- Duration: {time period}

## Success Criteria
- Primary: {metric} improves by ≥{threshold}%
- Secondary: No degradation in {guardrail metrics}
```

## Sample Size Calculation
```python
from statsmodels.stats.power import NormalIndPower

power = NormalIndPower()
n = power.solve_power(
    effect_size=0.2,    # Minimum detectable effect
    alpha=0.05,         # Significance level
    power=0.80          # Desired power
)
# n = number of samples per group
```

## Pitfalls
- **Peeking**: Don't stop test early when seeing results
- **Multiple comparisons**: Adjust significance for multiple metrics
- **Novelty effect**: Users may react differently to new features initially
- **Segmentation**: Test on full population, not just active users

## Verification
- Statistical significance reached (p < 0.05)
- Practical significance (is the effect size meaningful?)
- No negative impact on guardrail metrics
- Results replicated in second test?