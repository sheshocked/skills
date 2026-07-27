---
name: design-tokens-css
description: 
category: design
tags: [design-tokens-css]
---

## When to Use
Building a token-based design system: translating design decisions into code-ready variables. Bridges design and engineering when manual handoff creates drift.

## Core Concepts
- **Token taxonomy**: Global (primitive) → Semantic (alias) → Component (specific)
- **Token formats**: CSS custom properties, JSON, SCSS variables, Tailwind config
- **Mode/theme tokens**: Same token name, different values per theme (light/dark/high-contrast)
- **Token naming convention**: `--category-{property}-{variant}-{state}` (e.g., `--color-bg-primary-hover`)
- **Token inheritance**: Semantic tokens reference primitives; components reference semantics
- **Token tooling**: Style Dictionary, Figma Tokens plugin, Cobalt UI, Specify

## Workflow
1. Inventory existing hardcoded values across codebase (colors, spacing, font sizes)
2. Define primitive tokens: raw values organized by category
3. Create semantic tokens: map primitives to purpose-based names
4. Build component tokens: map semantics to component-specific names
5. Export tokens via Style Dictionary to CSS, SCSS, Tailwind, iOS, Android formats
6. Set up token build pipeline: Figma Tokens plugin → Style Dictionary → code
7. Implement theme switching via CSS custom property overrides on `:root` or `[data-theme]`
8. Document token usage with examples and visual swatches

## Key Patterns
- **Primitive → Semantic mapping**:
  ```css
  /* Primitive */
  --color-blue-500: #3b82f6;
  /* Semantic */
  --color-primary: var(--color-blue-500);
  /* Component */
  --button-bg: var(--color-primary);
  ```
- **Dark mode via data attribute**:
  ```css
  :root { --color-bg: #ffffff; --color-text: #1a1a1a; }
  [data-theme="dark"] { --color-bg: #0f172a; --color-text: #e2e8f0; }
  ```
- **Spacing scale (4px base)**:
  ```css
  --space-0: 0; --space-1: 0.25rem; --space-2: 0.5rem;
  --space-3: 0.75rem; --space-4: 1rem; --space-6: 1.5rem;
  --space-8: 2rem; --space-12: 3rem; --space-16: 4rem;
  ```
- **Style Dictionary config**:
  ```json
  { "source": ["tokens/*.json"],
    "platforms": { "css": { "transformGroup": "css", "buildPath": "dist/", "files": [{ "destination": "tokens.css", "format": "css/variables" }] } }
  ```

## Pitfalls
- Using primitive tokens directly in components — breaks theme switching; always use semantic tokens
- Inconsistent naming conventions — establish and enforce `category-variant-state` pattern
- Not automating token builds — manual copy-paste between Figma and code causes drift
- Too many tokens — every design decision doesn't need a token; tokens are for frequently-used, shared values
- Missing token documentation — engineers need to know what each token does and when to use it
- Ignoring spacing scale — arbitrary spacing values lead to inconsistent layouts

## Verification
- Run `grep -r "#[0-9a-fA-F]\{6\}" src/` — zero hardcoded hex values in component code
- Verify theme switching: toggle `[data-theme]` attribute, all UI updates without page reload
- Check token coverage: every CSS file uses only tokens, no raw values
- Test Style Dictionary output: CSS, SCSS, and JS exports all parse without errors