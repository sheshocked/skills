---
name: ui-ux-fundamentals
description: 
category: design
tags: [ui-ux-fundamentals]
---

## When to Use
Designing any user-facing interface from scratch or evaluating UX quality. Foundational knowledge needed before specialized UI work (mobile, dashboards, etc.).

## Core Concepts
- **Gestalt principles**: Proximity, similarity, closure, continuity, figure-ground
- **Hick's Law**: Decision time increases with number of choices
- **Fitts's Law**: Time to reach target = f(distance, size)
- **Jakob's Law**: Users prefer sites that work like ones they already know
- **Miller's Law**: Working memory holds ~7 items; chunk information accordingly
- **Information architecture**: How content is organized, labeled, and navigated

## Workflow
1. Define user goals (not business goals — translate business needs to user outcomes)
2. Create task flows: entry point → steps → success state → error states
3. Build low-fidelity wireframes to validate information hierarchy
4. Apply visual hierarchy: size, contrast, spacing, alignment guide the eye
5. Prototype key interactions before polishing visuals
6. User test with 5 users (Nielsen: 85% of issues found with n=5)
7. Iterate based on findings, not opinions

## Key Patterns
- **Visual hierarchy**: Heading 1 (24px bold) → Heading 2 (18px semibold) → Body (16px regular) → Caption (12px muted)
- **Progressive disclosure**: Show essential options first, reveal advanced settings on demand
- **Recognition over recall**: Use icons with labels, not icons alone; show recent items
- **Consistency**: Same action = same appearance everywhere (all primary buttons look identical)
- **Empty state design**: Every empty list should explain what goes there + how to add it

## Pitfalls
- Designing for yourself instead of users — validate assumptions with data or testing
- Ignoring error states — design the unhappy path before the happy path
- Using placeholder text ("Lorem ipsum") — real content reveals layout issues
- Over-relying on color for meaning — always pair with icons or text (accessibility)
- Skipping mobile considerations — design mobile-first, then expand

## Verification
- Perform a heuristic evaluation against Nielsen's 10 heuristics
- Time key tasks: can a new user complete core flow in < 60 seconds?
- Check color contrast ratios meet WCAG AA (4.5:1 text, 3:1 UI)
- Run a 5-user usability test and document findings