---
name: accessibility-design
description: 
category: design
tags: [accessibility-design]
---

## When to Use
Every design project — accessibility is not optional or a separate phase. Essential when designing for compliance (ADA, EAA, Section 508), but more importantly when designing for all users.

## Core Concepts
- **WCAG 2.2**: Level AA is the standard; Level AAA is aspirational
- **POUR principles**: Perceivable, Operable, Understandable, Robust
- **Focus management**: Visible focus indicators, logical tab order, focus trapping in modals
- **Screen reader compatibility**: ARIA roles, labels, live regions, landmark regions
- **Cognitive accessibility**: Clear language, consistent navigation, error prevention
- **Motor accessibility**: Large touch targets, keyboard alternatives, no time-dependent interactions
- **Color independence**: No information conveyed by color alone — add icons, patterns, or text

## Workflow
1. Start accessibility from wireframes, not as a final audit
2. Define semantic HTML structure before visual design
3. Design with sufficient contrast (WCAG AA: 4.5:1 text, 3:1 UI)
4. Create focus styles that are visible and match brand
5. Design form validation with inline errors, not just red borders
6. Include skip links, landmark regions, and heading hierarchy in layouts
7. Test with screen reader (VoiceOver/NVDA) during design review
8. Document ARIA patterns for complex components (modals, tabs, accordions)

## Key Patterns
- **Accessible form pattern**: Associated `<label>`, `aria-describedby` for hints, `aria-invalid` + `aria-errormessage` for errors
- **Focus indicator**: `outline: 2px solid var(--focus-ring); outline-offset: 2px;` — never `outline: none` without replacement
- **Modal focus trap**: Tab cycles within modal, Escape closes, focus returns to trigger element
- **Live region for dynamic content**: `<div aria-live="polite" aria-atomic="true">` for status updates
- **Skip navigation**: `<a href="#main" class="skip-link">Skip to content</a>` — visible on focus
- **Heading hierarchy**: h1 → h2 → h3, never skip levels (no h1 → h3)

## Pitfalls
- Adding ARIA to fix bad HTML — `role="button"` on a `<div>` instead of using `<button>`
- Color-only indicators — "required fields are red" fails without text/pattern alternative
- Auto-playing media without controls — violates WCAG 2.2.2 (Pause, Stop, Hide)
- Missing alt text on decorative images — use `alt=""` (empty) for decorative, descriptive alt for informative
- Modal that doesn't trap focus — keyboard users can tab behind the modal
- No error recovery guidance — "Invalid input" without telling user what's wrong or how to fix

## Verification
- Run axe DevTools or Lighthouse accessibility audit — zero critical/serious violations
- Navigate entire flow using only keyboard (Tab, Enter, Escape, Arrow keys)
- Test with VoiceOver (Mac) or NVDA (Windows) — all content announced correctly
- Check all form inputs have visible labels and associated error messages
- Validate focus order matches visual order
- Verify color contrast with WebAIM Contrast Checker for every text/background combo