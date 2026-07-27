---
name: interaction-motion-design
description: 
category: design
tags: [interaction-motion-design]
---

## When to Use
Adding purposeful animation to UI: page transitions, micro-interactions, loading states, feedback animations. When static designs feel lifeless or users miss state changes.

## Core Concepts
- **12 principles of animation** (Disney): anticipation, squash-stretch, follow-through apply to UI
- **Easing curves**: ease-out for entrances (fast start, slow end), ease-in for exits, ease-in-out for emphasis
- **Duration guidelines**: Micro-interactions 100-300ms, page transitions 300-500ms, loading > 500ms
- **Motion hierarchy**: Primary actions get prominent animation, secondary actions get subtle animation
- **Reduced motion**: `prefers-reduced-motion` media query — always respect user preference
- **Animation purpose**: Entering/exiting, emphasis, feedback, spatial orientation, progress

## Workflow
1. Identify where motion adds value (not decoration — must serve a function)
2. Define animation types: entrance, exit, emphasis, transition
3. Choose easing and duration per motion type
4. Prototype in code or Principle/After Effects for stakeholder review
5. Implement with CSS transitions/animations or Framer Motion/GSAP
6. Add `prefers-reduced-motion` fallbacks
7. Test on low-end devices — check for jank (target 60fps)
8. Review with team: does motion guide attention or distract?

## Key Patterns
- **Button hover micro-interaction** (CSS):
  ```css
  .btn { transition: transform 150ms ease-out, box-shadow 150ms ease-out; }
  .btn:hover { transform: translateY(-1px); box-shadow: 0 4px 12px rgba(0,0,0,0.15); }
  ```
- **Page transition with Framer Motion**:
  ```jsx
  <motion.div initial={{ opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }}
    exit={{ opacity: 0, y: -20 }} transition={{ duration: 0.3, ease: "easeOut" }} />
  ```
- **Skeleton loading → content reveal**: Fade skeleton out (200ms), fade content in (300ms), slight y-translate
- **Toast notification**: Slide in from top (300ms ease-out), auto-dismiss after 5s with progress bar
- **Reduced motion fallback**: Replace translate/scale with opacity-only transitions

## Pitfalls
- Animating everything — motion fatigue is real; only animate state changes that matter
- Using `ease-in-out` everywhere — it feels sluggish for entrances; use ease-out for things appearing
- Ignoring `prefers-reduced-motion` — violation of accessibility standards
- Long animations (>500ms) for common actions — feels slow even if technically smooth
- Not accounting for scroll position — fixed-position animations can feel disconnected
- Using `setTimeout` for sequencing — use CSS `animation-delay` or `transition-delay` for reliability

## Verification
- Check 60fps in Chrome DevTools Performance tab during all animations
- Toggle `prefers-reduced-motion: reduce` and verify all motion is replaced
- Test on a throttled CPU (4x slowdown) — animations should still feel intentional
- Verify no layout thrashing: animations only trigger `transform` and `opacity` (GPU-composited)