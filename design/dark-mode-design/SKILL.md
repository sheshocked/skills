---
name: dark-mode-design
description: 
category: design
tags: [dark-mode-design]
---

## When to Use
Implementing dark mode alongside light mode, or designing exclusively dark interfaces (developer tools, media players, creative apps). System-level dark mode support is now expected by users.

## Core Concepts
- **Dark mode ≠ inverted light mode**: Colors need individual adjustment, not just hue rotation
- **Elevation through brightness**: In dark mode, higher-elevation surfaces are lighter (opposite of light mode)
- **Color saturation**: Reduce saturation in dark mode — vivid colors on dark backgrounds cause eye strain
- **Text colors**: Use off-white (#e2e8f0) instead of pure white (#ffffff) — reduces glare
- **Contrast adaptation**: WCAG ratios must be maintained in BOTH themes — often need different color values
- **Surface hierarchy**: Background → surface → elevated surface → overlay (each progressively lighter)
- **System preference detection**: `prefers-color-scheme` media query for automatic switching

## Workflow
1. Build light mode first — it's the reference for contrast and hierarchy
2. Define dark mode palette: shift hues slightly, reduce saturation by 10-20%, adjust lightness
3. Map every semantic token to a dark mode equivalent
4. Adjust elevation: higher surfaces get lighter values (instead of darker shadows)
5. Replace shadows with subtle borders or lighter backgrounds in dark mode
6. Implement theme switching: CSS custom properties + `[data-theme]` or `prefers-color-scheme`
7. Test all components in both modes — modals, tooltips, dropdowns, overlays
8. Add toggle button with smooth transition between modes

## Key Patterns
- **Dark mode color mapping**:
  ```
  Light:  bg=#ffffff, surface=#f8fafc, border=#e2e8f0, text=#1a1a1a
  Dark:   bg=#0f172a, surface=#1e293b, border=#334155, text=#e2e8f0
  ```
- **Theme toggle implementation**:
  ```css
  :root { color-scheme: light; --bg: #ffffff; --text: #1a1a1a; }
  :root.dark { color-scheme: dark; --bg: #0f172a; --text: #e2e8f0; }
  @media (prefers-color-scheme: dark) {
    :root:not(.light) { color-scheme: dark; --bg: #0f172a; --text: #e2e8f0; }
  }
  ```
- **Image handling for dark mode**: Add subtle background, reduce brightness/contrast on images, or provide dark-mode variants
- **Shadow replacement**: In dark mode, use `box-shadow: 0 0 0 1px var(--border)` instead of `box-shadow: 0 4px 12px rgba(0,0,0,0.1)`
- **Color-adjust for print**: `@media print { color-scheme: light; }` — always print in light mode

## Pitfalls
- Pure black (#000000) background — causes eye strain and makes text hard to read; use `#0f172a` or `#121212`
- Not adjusting images — bright photos on dark backgrounds look jarring; darken or add padding
- Shadows invisible in dark mode — shadows on dark backgrounds are nearly invisible; use borders or opacity
- Brand colors that don't adapt — primary colors need dark variants for sufficient contrast
- Ignoring system preference — always detect `prefers-color-scheme` and respect user's OS setting
- Not transitioning theme change — abrupt color swap feels jarring; use `transition: background-color 200ms, color 200ms`

## Verification
- Toggle between light/dark mode — all text, icons, and interactive elements remain readable
- Check WCAG contrast in BOTH themes for every text/background combination
- Screenshot dark mode and compare to light mode: hierarchy should be equally clear
- Test with OS-level dark mode setting — app should auto-switch without manual toggle
- Verify images look good in both themes (no white halos, no invisible content)