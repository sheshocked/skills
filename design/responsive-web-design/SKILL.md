---
name: responsive-web-design
description: 
category: design
tags: [responsive-web-design]
---

## When to Use
Building websites that adapt across viewports — mobile phones, tablets, laptops, ultrawide monitors. Essential for any public-facing web product.

## Core Concepts
- **Mobile-first design**: Start at 320px, progressively enhance for larger screens
- **Breakpoint strategy**: Content-driven breakpoints, not device-driven (320, 640, 768, 1024, 1280, 1536)
- **Fluid grids**: CSS Grid / Flexbox with relative units (fr, %, rem) instead of fixed px
- **Container queries**: `@container` — respond to parent size, not viewport (component-level responsiveness)
- **Responsive images**: `srcset`, `<picture>` element, art direction for different crops
- **Intrinsic design**: Combine fluid (percentage-based) and fixed (content-sized) within one layout

## Workflow
1. Start with content inventory: what content exists and what's its priority at each size?
2. Design mobile layout first (single column, stacked content)
3. Define breakpoints where layout must change (not where device changes)
4. Build CSS Grid layouts with `auto-fit`/`auto-fill` for automatic reflow
5. Set up responsive typography with `clamp()` or breakpoint steps
6. Implement responsive images with art direction for key visuals
7. Test in browser DevTools at all breakpoints + physical devices
8. Audit: no horizontal scroll at any width, no content overlap, readable text

## Key Patterns
- **Fluid grid with auto-fit**:
  ```css
  .grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 1rem; }
  ```
- **Container query for card**:
  ```css
  .card-container { container-type: inline-size; }
  @container (min-width: 400px) { .card { flex-direction: row; } }
  ```
- **Responsive navigation**: hamburger menu below 768px, horizontal nav above
  ```css
  .nav-links { display: none; }
  @media (min-width: 768px) { .nav-links { display: flex; gap: 1rem; } }
  ```
- **Responsive image with art direction**:
  ```html
  <picture>
    <source media="(max-width: 640px)" srcset="hero-mobile.jpg" />
    <img src="hero-desktop.jpg" alt="..." />
  </picture>
  ```

## Pitfalls
- Pixel-based widths — `width: 1200px` breaks on tablets; use `max-width: 1200px` with fluid below
- Device-specific breakpoints — iPhone SE ≠ all small phones; design for content, not devices
- Hidden content on mobile — `display: none` means users miss it; reflow instead of hiding
- No touch-friendly spacing on desktop — generous hit areas help touchscreens and accessibility
- Fixed font sizes — always use rem/em so user zoom works
- Ignoring landscape orientation — test both orientations on phones and tablets

## Verification
- Chrome DevTools: test every breakpoint in the responsive panel
- Lighthouse: check for "Viewport meta tag" and "Document doesn't use legible font sizes"
- Resize from 320px → 2560px continuously — no content overlap, no horizontal scroll
- Check all images are responsive (no hardcoded width/height in px for images)
- Test with real devices: phone, tablet, laptop — physical testing catches issues simulators miss