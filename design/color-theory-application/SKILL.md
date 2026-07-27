---
name: color-theory-application
description: 
category: design
tags: [color-theory-application]
---

## When to Use
Selecting palettes for products, creating accessible color systems, or resolving color-related design inconsistencies. Critical for brand identity and UI consistency.

## Core Concepts
- **HSL color model**: Hue (0-360°), Saturation (0-100%), Lightness (0-100%) — most intuitive for design
- **Color harmony**: Complementary (180°), analogous (30°), triadic (120°), split-complementary
- **Perceptual uniformity**: OKLCH maintains consistent lightness across hue shifts (unlike HSL)
- **Contrast ratios**: WCAG AA requires 4.5:1 for normal text, 3:1 for large text/UI
- **Color semantics**: Colors carry meaning — red=danger, green=success, yellow=warning
- **Simultaneous contrast**: Colors appear different based on surrounding colors

## Workflow
1. Start with brand color(s) — extract HSL values
2. Generate a 50-950 scale using OKLCH for perceptual uniformity
3. Map semantic roles: primary, success, warning, error, info, neutral
4. Test all combinations for WCAG AA contrast (especially on dark/light backgrounds)
5. Define text colors: on-primary, on-surface, on-dark, on-light
6. Build CSS custom properties or design tokens
7. Validate in context: real UI mockups, not just swatches
8. Create dark mode variant by adjusting lightness, not just inverting

## Key Patterns
- **OKLCH color scale generation**: L goes 0.95 (50) → 0.15 (950), C peaks at 300-500 range, H shifts 10-20° across the scale
- **Semantic token mapping**:
  ```css
  --color-success: oklch(0.65 0.19 155);
  --color-success-light: oklch(0.93 0.06 155);
  --color-success-on: oklch(0.28 0.08 155);
  ```
- **Accessible pairings**: `#1a1a1a` on `#ffffff` = 16.6:1 ✓; `#767676` on `#ffffff` = 4.54:1 ✓; `#999999` on `#ffffff` = 2.85:1 ✗
- **Palette audit script**: Loop all background/foreground combos, flag anything below 4.5:1

## Pitfalls
- Using pure `#000000` on `#FFFFFF` — max contrast causes eye strain; use `#1a1a1a` on `#ffffff` instead
- Trusting HSL for lightness — two HSL colors with L=50 can look drastically different in brightness
- Hardcoding hex values in CSS — always use CSS custom properties or tokens
- Forgetting hover/focus states — need distinct color states for interactive elements
- Not testing with color-blindness simulators — ~8% of males have color vision deficiency

## Verification
- Run `a11y-audit` or Lighthouse contrast check on all text elements
- Screenshot under Deuteranopia, Protanopia, Tritanopia filters (Chrome DevTools)
- Verify every color in the system has both light and dark background variants
- Check that no meaning is conveyed by color alone