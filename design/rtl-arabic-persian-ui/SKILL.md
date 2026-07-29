---
name: rtl-arabic-persian-ui
description: Configure Vazirmatn typography, mirror structural layouts, and format Persian numbers dynamically.
category: design
tags: [rtl, layout-mirroring, vazirmatn, persian-ui, arabic-ui, web-design]
---

# RTL Persian/Arabic Layout Mirroring Masterclass

## When to Use
Use when formatting user interfaces targeting Persian/Arabic regions, ensuring layouts flow naturally from right to left, fonts align properly, and numbers format correctly.

## Prerequisites
- Shabnam or Vazirmatn font files.

## Workflow
1. Apply the HTML `dir="rtl"` attribute globally.
2. Mirror navigation positions, layouts, and icons dynamically.
3. Localize numerals configurations.

## Key Patterns

### Global RTL CSS Variables
```css
/* vazirmatn font mapping */
@font-face {
  font-family: 'Vazirmatn';
  src: url('/fonts/Vazirmatn-Regular.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
}

:root {
  font-family: 'Vazirmatn', system-ui, -apple-system, sans-serif;
}

/* Logical properties prevent hardcoded directional limits */
.my-card {
  padding-inline-start: 1.5rem; /* Mirrors padding dynamically */
  text-align: start;            /* Adjusts to right-align automatically */
}
```

### Persian Numbers Formatter Script
```javascript
// Localize numbers statically to Persian representation
export function toPersianDigits(num) {
  if (num === null || num === undefined) return "";
  const persianDigits = ["۰", "۱", "۲", "۳", "۴", "۵", "۶", "۷", "۸", "۹"];
  return num.toString().replace(/\d/g, (x) => persianDigits[parseInt(x)]);
}

// Verification output: toPersianDigits(185) -> "۱۸۵"
```

## Pitfalls
- **Static padding styles:** Using `margin-left` or `padding-right` instead of `margin-inline-start` breaks spacing when switching directions.
- **Mirroring wrong icons:** Do not mirror universal icons (e.g. settings gears, information icons). Mirror directional ones (e.g. forward/backward arrows).

## Verification
- Toggle `dir="rtl"` in inspector and verify no components overlap.
- Check font rendering behavior on small screen mobile viewports.
