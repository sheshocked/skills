---
name: rtl-arabic-persian-ui
description: Configure RTL layout mirroring, Vazirmatn font-family, and Persian numbers formatting.
category: design
tags: [rtl, persian-ui, arabic-layout, vazirmatn, css]
---

# Rtl Arabic Persian Ui

## When to Use
Use when formatting web user interfaces targeted at Persian/Arabic speakers to mirror layouts and handle localized text wraps.

## Prerequisites
- Persian fonts (Vazirmatn / Shabnam).

## Workflow
1. Apply `dir="rtl"` to HTML elements.
2. Mirror spatial layouts: use logical properties (`ms-*`, `me-*`) instead of directional ones (`ml-*`, `mr-*`).
3. Bind Vazimatn fonts to UI.

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
- **Directional arrow icons:** Ensure navigation icons (e.g. back button arrows) flip direction dynamically in RTL.
- **Numbers formatting:** Format data variables to local strings: `num.toLocaleString('fa-IR')` to display Persian digits.

## Verification
- Inspect layout boxes and confirm alignment changes when toggling `dir="rtl"`.
- Test rendering of Persian text wraps on small device displays.
