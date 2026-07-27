---
name: pricing-page-design
description: 
category: design
tags: [pricing-page-design]
---

## When to Use
Designing pricing pages for SaaS, marketplaces, or subscription products. The page must communicate plan differences, build trust, and drive the highest-value conversion.

## Core Concepts
- **Price anchoring**: Show the highest price first to make mid-tier feel reasonable
- **Decoy effect**: A poorly-valued third option makes the target plan more attractive
- **Feature comparison**: Clear differentiation between plans — what you get at each level
- **Social proof integration**: "Most popular" badges, customer counts, testimonials near pricing
- **Annual vs monthly toggle**: Annual pricing with savings percentage drives commitment
- **Trust signals**: Money-back guarantee, free trial, no credit card required, cancel anytime
- **Enterprise/Contact Sales**: For high-value customers who need custom pricing

## Workflow
1. Define 3 tiers: Starter/Pro/Enterprise (or Free/Pro/Team)
2. Determine the "anchor" — the tier you most want to sell (usually mid-tier)
3. Design tier comparison: feature matrix with clear ✗/✓/unlimited markers
4. Add annual/monthly toggle with savings callout
5. Place CTAs strategically: "Start free trial" for self-serve, "Talk to sales" for enterprise
6. Build FAQ section addressing common pricing objections
7. Add trust elements: security badges, compliance logos, customer logos
8. A/B test: price presentation, feature ordering, CTA copy

## Key Patterns
- **Three-tier layout**:
  ```
  [Starter]        [Pro ★ most popular]     [Enterprise]
  $9/mo            $29/mo                    Custom
  5 projects       Unlimited projects        Unlimited + SSO
  Basic features   Advanced features         Everything + support
  Email support    Priority support          Dedicated account manager
  [Start free]     [Start free trial]        [Talk to sales]
  ```
- **Feature comparison table**:
  ```
  | Feature            | Starter | Pro  | Enterprise |
  |--------------------|---------|------|------------|
  | Projects           | 5       | ∞    | ∞          |
  | Team members       | 3       | 20   | Unlimited  |
  | SSO/SAML           | ✗       | ✗    | ✓          |
  | Custom domain      | ✗       | ✓    | ✓          |
  ```
- **Annual savings badge**: "Save 20%" in green pill badge next to annual price
- **Money-back guarantee bar**: "30-day money-back guarantee. No questions asked." below pricing cards
- **Comparison with competitor**: "See how we compare" section with honest feature comparison

## Pitfalls
- Too many tiers (>4) — causes decision paralysis; 3 is optimal
- Hidden pricing — requiring email or demo for basic plans reduces trust
- Feature lists that are identical — tiers must have clear, meaningful differences
- Annual-only pricing — alienates users who want to try before committing
- No free tier or trial — most SaaS expects try-before-buy
- Ignoring mobile — pricing cards must stack cleanly on small screens
- CTA below the fold — "Start free trial" button must be visible without scrolling on desktop

## Verification
- 5-second test: can users identify which plan is recommended and what it costs?
- Mobile check: pricing cards stack vertically, all features visible, CTAs accessible
- A/B test: run experiments on pricing display, feature order, and CTA copy
- Conversion funnel: track visit → plan selection → trial start → paid conversion
- User testing: ask 5 users "which plan would you choose and why?" — answers should be clear