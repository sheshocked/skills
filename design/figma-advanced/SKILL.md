---
name: figma-advanced
description: 
category: design
tags: [figma-advanced]
---

## When to Use
Building complex Figma workflows: auto-layout systems, component variants, prototyping, dev handoff, and plugin automation. When basic Figma usage hits limits.

## Core Concepts
- **Auto Layout**: Flexbox-equivalent in Figma — direction, padding, gap, alignment, wrapping
- **Component variants**: Properties panel for boolean/color/instance swap variants
- **Nested instances**: Override child components within a parent instance
- **Design tokens plugin**: Export colors, typography, spacing to JSON/CSS variables
- **Branching & merging**: Team collaboration with conflict resolution
- **Variables**: Figma's native design tokens (color, number, string, boolean)
- **Dev Mode**: Spec-ready measurements, CSS/code snippets for engineers

## Workflow
1. Set up team library: publish foundational components (buttons, inputs, cards)
2. Use auto-layout on every component — no manual positioning
3. Define variants: size × state × content combos (e.g., Button[primary/sm/disabled])
4. Build page templates using nested components and instances
5. Create interactive prototypes with smart animate for key flows
6. Use variables for tokens (color, spacing) — switch themes via modes
7. Share via Dev Mode for engineering handoff
8. Maintain: archive stale pages, update components quarterly

## Key Patterns
- **Auto-layout card component**:
  ```
  Frame (auto-layout, vertical, gap: 16, padding: 24)
  ├── Image (fill width, aspect-ratio: 16/9)
  ├── Text group (auto-layout, vertical, gap: 4)
  │   ├── Title (heading, fill width)
  │   └── Description (body, fill width)
  └── Button row (auto-layout, horizontal, gap: 8, trailing)
  ```
- **Variant property setup**: Boolean props (`loading`, `disabled`), Instance Swap (`icon`), Enum (`size: sm | md | lg`)
- **Figma Variables for theming**: Create modes (Light, Dark) in a collection; swap entire UI in one click
- **Component documentation**: Add description field + link to usage guidelines in the component's description

## Pitfalls
- Not using auto-layout — manual positioning breaks responsiveness and slows iteration
- Too many variant combinations — use only what's needed; merge rarely-used states into boolean props
- Publishing broken components to team library — always test with a fresh file before publishing
- Ignoring component description fields — engineers depend on them in Dev Mode
- Not using auto-naming (`/` separator) — names like `Button / Primary / Large` auto-populate variant properties

## Verification
- Select any instance → check overrides panel shows meaningful variant selections
- Run a component audit: every published component has a description, variants, and usage example
- Test responsive behavior: resize parent frame, verify auto-layout adapts correctly
- Hand off to an engineer: can they implement the component from Dev Mode specs alone?