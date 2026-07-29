---
name: figma-to-code
description: Translate complex layout layers, auto-layout frames, grid variables, and styling tokens from Figma to CSS.
category: design
tags: [figma, tailwind-css, design-tokens, layout-mapping, web-design]
---

# Figma To Code

## When to Use
Use when converting professional UI layout designs from Figma into pixel-perfect frontend code components using utility classes.

## Prerequisites
- Access to Figma design layers.

## Workflow
1. Map Figma typography hierarchy variables directly to CSS theme declarations.
2. Convert Figma Auto Layout constraints to flexbox grids (`flex`, `grid`, `gap`).
3. Extract spacing tokens to Tailwind classes mapping.

## Key Patterns

### Auto Layout constraints translation:
- Auto Layout **Vertical** (with gap: 16px) ──> class `flex flex-col gap-4`
- Auto Layout **Horizontal** (with alignment: Space Between) ──> class `flex flex-row justify-between items-center`
- Fixed constraints (padding: 24px) ──> class `p-6`

### Figma spacing mapping config (tailwind.config.js)
```javascript
module.exports = {
  theme: {
    spacing: {
      'xs': '4px',
      'sm': '8px',
      'md': '16px',
      'lg': '24px',
      'xl': '32px',
    }
  }
}
```

## Pitfalls
- **Ignoring line heights:** Figma defaults line heights to auto, whereas browsers apply local overrides. Always map line heights explicitly in CSS to prevent layout breaks.
- **Absolute coordinates:** Avoid copying absolute positioning `left: 20px; top: 10px` coordinates. Always use responsive layout variables.

## Verification
- Verify code output matches design visual aspects under varying screen widths.

