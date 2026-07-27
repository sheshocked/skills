---
name: design-critique
description: 
category: design
tags: [design-critique]
---

## When to Use
Running or participating in design reviews: structured feedback sessions that improve work quality without ego or politics. Essential for team-based design processes.

## Core Concepts
- **Critique ≠ criticism**: Analysis of work against goals, not personal judgment
- **SBI feedback model**: Situation (context), Behavior (what you observe), Impact (effect on user)
- **Design principles as filter**: Feedback should reference agreed-upon principles, not personal taste
- **Roles**: Presenter shares context + questions; Critiquer analyzes + suggests; Facilitator keeps time/flow
- **Hamburger protocol**: Something positive → constructive concern → positive closer
- **Async critique**: Written feedback via shared doc or Figma comments — often more thoughtful than live

## Workflow
1. Presenter prepares: brief context (goals, constraints, research), specific questions, work-in-progress
2. Share materials 24h before session for async review
3. Facilitator sets ground rules: reference design principles, focus on user impact, suggest alternatives
4. Presenter gives 2-minute walkthrough (no defending — listen only)
5. Critiquers ask clarifying questions first ("What problem does this solve?")
6. Feedback round: each person shares observations using SBI model
7. Presenter summarizes: what resonates, what to explore further, decisions to make
8. Document decisions and action items; follow up within 48 hours

## Key Patterns
- **Critique prompt card**:
  ```
  1. What problem is this design solving?
  2. Who is the user and what are their goals?
  3. What are the key constraints?
  4. What decisions do you need feedback on?
  5. What's still uncertain?
  ```
- **SBI feedback example**:
  ```
  "In the checkout flow (Situation), the form has 8 fields on one screen (Behavior),
  which will likely increase abandonment — research shows > 5 fields reduces completion by 20% (Impact).
  Consider splitting into 2 steps or using progressive disclosure."
  ```
- **Principle-based critique**:
  ```
  "Our design principle says 'clarity over cleverness.' This animation is clever but
  delays the user from seeing their result. Can we simplify?"
  ```
- **Async critique template**:
  ```
  Context: [what/why]
  Questions: [specific asks]
  What I'm unsure about: [areas needing validation]
  Please comment directly on the Figma file.
  ```
- **Decision matrix after critique**:
  ```
  | Feedback point | Agreed? | Owner | Due date |
  |----------------|---------|-------|----------|
  | Simplify form  | Yes     | Ali   | Friday   |
  | New icon set   | Discuss | Sara  | Monday   |
  ```

## Pitfalls
- Critiquing without context — "I don't like this color" without understanding goals is useless
- Presenting finished work — critique is most valuable at early, exploratory stages
- Not separating exploration from decision — clarify whether you want ideas or sign-off
- Attacking the person instead of the work — "This is confusing" not "You made this confusing"
- Skipping the positive — always acknowledge what works before addressing concerns
- No follow-through — critique without documented decisions and owners is wasted time

## Verification
- Every critique session produces documented decisions with owners and deadlines
- Feedback references design principles or user research, not personal preference
- 80%+ of critique feedback includes a suggested alternative (not just "this is wrong")
- Team retrospectives show critique sessions are rated as valuable by all participants
- Design quality measurably improves after critique (fewer revision rounds, fewer bugs)