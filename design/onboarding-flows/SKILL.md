---
name: onboarding-flows
description: 
category: design
tags: [onboarding-flows]
---

## When to Use
Designing first-time user experiences: welcome screens, tutorials, setup wizards, progressive onboarding. When reducing time-to-value and activation is a product priority.

## Core Concepts
- **Time-to-first-value**: Minimize steps between signup and user achieving their goal
- **Progressive onboarding**: Teach features in context when the user first encounters them
- **Activation milestone**: The moment a user completes a key action that predicts retention
- **Onboarding patterns**: Self-serve walkthrough, empty state coaching, contextual tooltips, checklist
- **Progress indicators**: Show completion percentage or step count to motivate continuation
- **Skip option**: Always let experienced users skip — forced onboarding causes abandonment
- **Aha moment**: Identify the single action most correlated with retention and optimize toward it

## Workflow
1. Define the activation milestone: what action must users take to "get it"?
2. Map the minimum steps from signup to activation (aim for ≤ 3 steps)
3. Design welcome screen: greet, state value prop, single CTA to begin
4. Build setup wizard: collect only essential info needed to start (name, use case, goal)
5. Design contextual onboarding: tooltips, coach marks, or inline hints at key features
6. Create onboarding checklist: visible progress toward activation
7. Instrument analytics: track drop-off at each step, time-to-activation
8. Iterate: A/B test step count, copy, and timing of tooltips

## Key Patterns
- **Setup wizard structure**:
  ```
  Step 1: Welcome → "Let's set up your workspace" (CTA: Get started)
  Step 2: Essential info → Name, team size, primary use case (2 fields max)
  Step 3: First action → Create first project/item with guided template
  Step 4: Success → "You're all set! Here's what you can do next..."
  ```
- **Contextual tooltip**:
  ```
  Position: below the element (or above if near bottom)
  Content: "This is where your [feature] lives. Click to [action]."
  Dismiss: "Got it" button + "Don't show again" link
  Trigger: First time element is visible
  ```
- **Onboarding checklist**:
  ```
  ☐ Create your first project (required for activation)
  ☐ Invite a team member
  ☐ Connect your first integration
  ☐ Complete your profile
  Progress: 1/4 complete — "You're on your way!"
  ```
- **Empty state as onboarding**:
  ```
  "Your dashboard is empty"
  "Start by creating your first [item]" → [Create button]
  Show a 30-second video walkthrough
  ```

## Pitfalls
- Multi-page tutorial before user can do anything — users want to act, not watch
- Collecting too much info upfront — every extra field reduces completion rate by ~10%
- No skip option — power users and returning users should bypass onboarding
- Tooltips that don't dismiss — overlapping tooltips frustrate users; show one at a time
- Not measuring activation — if you don't track it, you can't improve it
- Generic onboarding for all users — different user types need different first experiences

## Verification
- Time-to-activation: measure median time from signup to activation milestone (target: < 5 minutes)
- Onboarding completion rate: > 80% of users should complete setup wizard
- Drop-off analysis: identify the step with highest abandonment and optimize it
- User test with 5 new users: can they reach activation without external help?
- Analytics: track onboarding funnel (signup → step1 → step2 → ... → activation)