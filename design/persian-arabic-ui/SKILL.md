---
name: persian-arabic-ui
description: 
category: design
tags: [persian-arabic-ui]
---

## When to Use
Designing interfaces for Persian (Farsi) or Arabic languages: RTL layout, bidirectional text, culturally appropriate UI patterns, and type design for Arabic script.

## Core Concepts
- **RTL (Right-to-Left)**: Layout mirrors horizontally — content flows right-to-left, alignment flips
- **Bidirectional (BiDi) text**: Numbers and Latin text within RTL context still read left-to-right
- **CSS logical properties**: `margin-inline-start` instead of `margin-left` — auto-flips for RTL
- **Arabic/Persian typography**: Connected script, baseline-driven, requires appropriate font metrics
- **Mirroring rules**: UI elements that imply direction (arrows, progress bars, timelines) must flip; others (play button, checkmarks) must not
- **Number systems**: Persian uses ۰۱۲۳۴۵۶۷۸۹; Arabic uses ٠١٢٣٤٥٦٧٨٩; both use Western digits in tech contexts
- **Text expansion**: RTL languages can be 30-40% longer than English — design with flexible containers

## Workflow
1. Set `dir="rtl"` and `lang="fa"` (or `lang="ar"`) on the root HTML element
2. Replace all directional CSS properties with logical equivalents (margin-left → margin-inline-start)
3. Mirror layout: navigation, sidebars, content flow all flip horizontally
4. Select Arabic/Persian font with proper metrics (e.g., Vazirmatn, IBM Plex Sans Arabic, Tajawal)
5. Test bidirectional text: mixed Arabic + numbers + English URLs display correctly
6. Verify icon mirroring: arrows, chevrons, progress indicators flip; logos, play/pause don't
7. Test with RTL pseudo-locale (`<html dir="rtl" lang="ar">` on English content) before translation
8. Validate with native speakers for layout, typography, and cultural appropriateness

## Key Patterns
- **CSS logical properties**:
  ```css
  /* Instead of: margin-left: 16px; padding-right: 24px; text-align: right; */
  .card { margin-inline-start: 16px; padding-inline-end: 24px; text-align: start; }
  ```
- **Mirroring with data attribute**:
  ```css
  [dir="rtl"] .icon-arrow { transform: scaleX(-1); }
  [dir="rtl"] .icon-play { /* NO flip */ }
  [dir="rtl"] .sidebar { margin-inline-start: auto; /* right side in RTL */ }
  ```
- **Flexbox auto-mirrors**: `justify-content: flex-start` automatically becomes `flex-end` in RTL — no extra CSS needed
- **Grid layout mirroring**: `grid-template-columns: 200px 1fr` in LTR becomes `1fr 200px` in RTL — logical properties handle it
- **Font stack for Arabic**:
  ```css
  :root { --font-family: "Vazirmatn", "IBM Plex Sans Arabic", "Segoe UI", Tahoma, sans-serif; }
  ```

## Pitfalls
- Hardcoded `left`/`right` in CSS — breaks in RTL; always use `inline-start`/`inline-end`
- Not mirroring navigation — sidebar on left in LTR must move to right in RTL
- Using `text-align: right` instead of `text-align: end` — breaks in bidirectional contexts
- Ignoring number direction — phone numbers, dates, and codes should remain LTR within RTL text
- Choosing wrong font — Latin-optimized fonts have poor Arabic glyph rendering; use Arabic-first or bilingual fonts
- Not testing text expansion — RTL translations are longer; containers must accommodate 40% growth
- Forgetting to mirror the scrollbar — some browsers don't auto-mirror scrollbar position

## Verification
- Switch `dir="rtl"` on the entire page — layout mirrors completely, no broken elements
- Check all interactive elements: tabs, carousels, sliders work in reverse direction
- Verify BiDi text: Arabic text with numbers and URLs renders correctly (numbers LTR within RTL flow)
- Test with actual Persian/Arabic content (not Lorem ipsum) — check line breaks, word wrapping
- Check CSS: `grep -r "margin-left\|margin-right\|padding-left\|padding-right" src/` should return zero results