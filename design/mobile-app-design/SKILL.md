---
name: mobile-app-design
description: 
category: design
tags: [mobile-app-design]
---

## When to Use
Designing native mobile apps (iOS/Android) or mobile-first responsive apps. When platform-specific conventions, touch interactions, and mobile constraints are primary concerns.

## Core Concepts
- **Platform guidelines**: Apple HIG (iOS), Material Design 3 (Android) — know both even if building for one
- **Touch targets**: Minimum 44×44pt (iOS) / 48×48dp (Android) for interactive elements
- **Safe areas**: Respect notch, Dynamic Island, and system gesture zones
- **Bottom navigation vs tabs**: Bottom nav for 3-5 top-level destinations; tab bar for same-level content
- **Thumb zones**: Most-used actions should be within easy thumb reach (bottom 60% of screen)
- **Offline-first**: Design for degraded connectivity — cache, retry, show stale data with indicator
- **Platform conventions**: Pull-to-refresh, swipe actions, long-press context menus

## Workflow
1. Define platform targets (iOS, Android, or both via cross-platform)
2. Map user flows → identify which screens need custom design vs. platform patterns
3. Design at 1x (375pt iPhone) and test at 3x (iPhone Plus/Max)
4. Use platform components where possible (navigation bars, modals, sheets)
5. Design all states: loading, empty, error, offline, success, edge cases
6. Create interactive prototype with realistic gestures (swipe, long-press)
7. Test on real devices — simulator layouts miss real-world thumb reach
8. Prepare assets: SVG icons, adaptive icons (Android), app icon variations

## Key Patterns
- **Bottom sheet pattern**: Drag handle + scrollable content + backdrop. Use for secondary actions without navigation
- **Pull-to-refresh**: Circular progress indicator that triggers on overscroll release
- **Swipe actions on list items**: Reveal destructive (red, left-swipe) or contextual actions
- **Skeleton screens**: Show layout structure during loading instead of spinners
- **Haptic feedback mapping**: Light tap (selection), medium (action), heavy (destruction)
- **Adaptive layout**: `max-width: 428px` centered on tablets, full-bleed on phones

## Pitfalls
- Treating mobile like desktop with smaller viewport — mobile has fundamentally different interaction patterns
- Ignoring the keyboard — input fields must scroll into view and remain accessible
- Bottom sheet + modal stacking — users get confused by nested overlays
- Tiny tap targets to fit more on screen — violates platform minimums, causes frustration
- Not designing for notched devices — content gets clipped in safe areas
- Ignoring dark mode — system-wide dark mode is expected on both platforms

## Verification
- Test touch target sizes with accessibility inspector (iOS) / Accessibility Scanner (Android)
- Run on 3+ device sizes: small phone (iPhone SE), standard (iPhone 15), large (iPhone 15 Pro Max)
- Verify all interactive elements are reachable with one-handed thumb use
- Check offline behavior: airplane mode test for every data-dependent screen
- Validate with platform design review checklist (iOS HIG / Material guidelines)