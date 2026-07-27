---
name: typography-systems
description: 
category: design
tags: [typography-systems]
---

## When to Use
Establishing type scales, font selections, and typographic hierarchies for products. Needed when building design systems, creating brand guidelines, or fixing inconsistent text rendering across platforms.

## Core Concepts
- **Type scale**: Mathematical progression of font sizes (common: Major Third 1.25, Perfect Fourth 1.333)
- **Modular scale**: Base size × ratioⁿ for each step (e.g., 16px, 20px, 25px, 31px at 1.25 ratio)
- **Line height**: 1.4-1.6 for body text, 1.1-1.2 for headings
- **Measure**: Optimal line length is 45-75 characters (roughly 65ch)
- **Vertical rhythm**: Consistent spacing baseline aligned to line-height
- **Variable fonts**: Single file with axis-variation for weight, width, optical size

## Workflow
1. Choose primary typefaces (max 2: one for headings, one for body; or one variable font)
2. Define base font size (16px for web, 16sp for Android, 17pt for iOS)
3. Calculate modular scale from base (e.g., 16 × 1.25ⁿ)
4. Set line-height per size (smaller text = more generous leading)
5. Define tracking (letter-spacing) adjustments for caps/small text
6. Build type tokens: font-family, font-size, font-weight, line-height, letter-spacing
7. Create responsive type: fluid sizing with `clamp()` or breakpoint steps
8. Test rendering across browsers, OS, and screen densities

## Key Patterns
- **Fluid type with clamp()**:
  ```css
  --font-h1: clamp(1.5rem, 0.8rem + 3vw, 3rem);
  ```
- **Vertical rhythm via baseline grid**:
  ```css
  :root { --line-height: 1.5; --baseline: calc(1rem * var(--line-height)); }
  p { margin-bottom: var(--baseline); }
  ```
- **Type scale (Perfect Fourth)**:
  ```
  xs: 12px (0.75rem), sm: 14px, base: 16px, lg: 21px, xl: 28px, 2xl: 38px, 3xl: 50px
  ```
- **Font loading strategy**: `font-display: swap` + preload critical weights, lazy-load decorative

## Pitfalls
- Using more than 2 font families — creates visual noise and slows loading
- Setting `line-height` without units (unitless = relative, px = absolute — use unitless)
- Ignoring font rendering across platforms — hinting differs on Windows vs macOS
- Missing italic/bold fallbacks — always define fallback stack
- Hardcoding font sizes in px — use rem/em for user zoom support
- Variable font weight range too narrow — ensure enough contrast between headings and body

## Verification
- Check all text meets WCAG contrast ratios
- Verify vertical rhythm: measure baseline grid alignment in browser dev tools
- Test at 200% zoom: text must reflow without horizontal scrolling
- Compare rendering: Chrome, Firefox, Safari, Edge (font-smoothing varies)
- Validate font loading: no FOUT/FOIT on slow connections