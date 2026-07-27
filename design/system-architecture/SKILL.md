---
name: system-architecture
description: 
category: design
tags: [system-architecture]
---

## When to Use
Building or scaling a design system: establishing shared components, tokens, patterns, and documentation across product teams. Essential when 3+ products share a brand or when design debt causes inconsistency.

## Core Concepts
- **Design tokens**: Atomic visual values (colors, spacing, typography) as variables
- **Component API**: Props, variants, and composition rules for reusable UI elements
- **Governance model**: Who proposes, reviews, and accepts system changes
- **Figma ↔ Code sync**: Keeping design files aligned with shipped components
- **Versioning**: Semantic versioning for breaking vs. additive changes

## Workflow
1. Audit existing products for repeated patterns (buttons, inputs, modals)
2. Define token primitives: colors, spacing scale, type scale, radii, shadows
3. Build component library in code (React/Vue/Web Components)
4. Mirror components in Figma with auto-layout and variants
5. Document usage guidelines, do/don't examples, accessibility notes
6. Set up change process: RFC → review → merge → publish
7. Track adoption metrics: % of screens using system components

## Key Patterns
- **Token hierarchy**: primitive → semantic → component-level. Example: `blue-500` → `color-primary` → `button-background`
- **Component variants via props**: `<Button variant="primary" size="lg" disabled />`
- **Composition over configuration**: Prefer `<Card><CardHeader/><CardBody/></Card>` over 15 boolean props
- **Figma branching**: Feature branches for new components, merge after code ships

## Pitfalls
- Building components nobody uses — solve real pain points first, don't design in abstract
- Over-abstracting too early — start with 3 concrete use cases before generalizing
- Skipping accessibility — every component must meet WCAG 2.1 AA minimum
- No versioning discipline — breaking changes without migration guides destroy trust
- Figma and code drift — automate sync checks in CI

## Verification
- Run visual regression tests (Chromatic, Percy) against component library
- Check component API docs are generated from code, not hand-written
- Audit: every screen in Figma uses only system tokens, no hardcoded values
- Measure adoption: `grep -r "from '@design-system'" src/ | wc -l`