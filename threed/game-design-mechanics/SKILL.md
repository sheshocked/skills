---
name: game-design-mechanics
description: - Balancing difficulty curves and player progression
category: threed
tags: [game-design-mechanics]
---

## When to Use
- Designing core gameplay loops (action, progression, reward systems)
- Balancing difficulty curves and player progression
- Creating engaging feedback systems (juice, game feel)
- Prototyping and iterating on mechanic interactions
- Documenting game design for teams (GDDs)

## Core Concepts
- Core Loop: what the player does repeatedly (explore → fight → loot)
- Game Feel (juice): screen shake, particles, sound, delay on hit
- Progression Systems: XP curves, skill trees, equipment upgrades
- Risk/Reward: difficulty vs payoff balance
- Flow State: challenge matches player skill (Csikszentmihalyi)
- MDA Framework: Mechanics → Dynamics → Aesthetics
- Emergent Gameplay: systems interacting to create unexpected behaviors

## Workflow
1. Define the core loop on paper before touching any tool
2. Prototype the loop in graybox (placeholder art, real mechanics)
3. Playtest with 3-5 people, observe frustration points
4. Iterate: adjust timing, difficulty, reward frequency
5. Add juice (feedback) to make actions feel impactful
6. Document in GDD with specific numbers, not vague descriptions
7. Validate with analytics: are players completing the loop?

## Key Patterns
```
# Difficulty curve formula (exponential scaling)
difficulty = baseDifficulty * (1.0 + growthRate) ^ level

# XP to next level (diminishing returns)
xp_required(level) = floor(100 * level^1.5)

# Health/damage formula for balance
effective_damage = attack * (100 / (100 + defense))
# Defense of 50 = 33% damage reduction
# Defense of 100 = 50% damage reduction (diminishing)

# Loot table with weighted drops
LootTable {
  Common:    weight=60 → small health potion
  Uncommon:  weight=25 → medium health potion
  Rare:      weight=10 → rare weapon
  Legendary: weight=5  → legendary armor
}
Total weight = 100; roll random, subtract weights until hit

# Game feel timing (ms)
Hit stop:        50-100ms
Screen shake:    100-200ms
Damage number:   200-400ms fade
Input buffer:    80-120ms (for action games)
```

GDD template structure:
```
1. Overview (1 page, elevator pitch)
2. Core Loop (diagram)
3. Mechanics (each: trigger → response → feedback → consequence)
4. Progression (XP curves, unlock order)
5. Economy (currency sources/sinks, balance sheet)
6. Controls (input → action mapping)
7. UI/UX (screens, flow diagram)
8. Monetization (if applicable, F2P vs premium)
```

## Pitfalls
- Over-designing before prototyping: mechanics that look good on paper fail in practice
- Difficulty spikes cause quit rates: smooth curves with checkpoints
- Too much reward too early: inflation, no motivation to continue
- Ignoring onboarding: 80% of players quit in first 5 minutes
- Scope creep: "one more feature" kills timelines — cut to core loop first

## Verification
- Playtest with strangers (not just your team) for unbiased feedback
- Track retention curves: D1, D7, D30 targets
- A/B test mechanic changes with analytics instrumentation
- Balance spreadsheet: simulate 1000 playthroughs with automated bots
- Review similar games' progression data for benchmarking