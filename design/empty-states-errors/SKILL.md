---
name: empty-states-errors
description: 
category: design
tags: [empty-states-errors]
---

## When to Use
Designing moments when there's nothing to show (empty lists, first use) or something went wrong (errors, timeouts, offline). These states are often neglected but heavily impact user perception and retention.

## Core Concepts
- **Empty state types**: First-use (onboarding), no-results (search), no-data (new account), cleared (user action)
- **Error state types**: Inline validation (form fields), page-level (404, 500), network (offline, timeout)
- **Error recovery**: Always tell users what happened AND what they can do next
- **Illustration style**: Match brand personality — friendly/doodle for casual apps, minimal/abstract for enterprise
- **Progressive disclosure**: Show just enough to orient, reveal details on demand
- **State persistence**: Preserve user's work when possible (localStorage, optimistic updates)

## Workflow
1. Inventory all empty/error states in the application (use state mapping)
2. Categorize: is this preventable? recoverable? permanent?
3. Design illustration/icon for each state type (consistent style)
4. Write microcopy: what happened (1 line), why it matters (1 line), what to do next (CTA)
5. Build components: EmptyState, ErrorBoundary, Toast, InlineError, OfflineBanner
6. Implement graceful degradation: cached data → skeleton → empty state → error
7. Add retry mechanisms for transient errors (network, server)
8. Test: deliberately trigger every error state and verify recovery path

## Key Patterns
- **Empty list with action**:
  ```
  Icon: (relevant illustration, 80-120px)
  Title: "No projects yet"
  Description: "Create your first project to start tracking progress."
  CTA button: "Create project"
  ```
- **Inline form error**:
  ```html
  <input aria-invalid="true" aria-describedby="email-error" />
  <span id="email-error" role="alert">Please enter a valid email address (e.g., name@example.com)</span>
  ```
- **404 page with personality**:
  ```
  Illustration: lost/confused character
  Title: "Page not found"
  Description: "The page you're looking for doesn't exist or was moved."
  Links: [Go home] [Search] [Contact support]
  ```
- **Offline banner**:
  ```
  Fixed top bar: "You're offline. Changes will sync when connection returns."
  Background: subtle warning color. Dismissible only when online returns.
  ```
- **Optimistic UI**: Show the result immediately (item added to list), revert if API fails

## Pitfalls
- Generic "Something went wrong" with no recovery path — users feel helpless
- Showing stack traces or technical errors to users — always show human-readable messages
- Not preserving user input on error — form data lost after validation failure is infuriating
- Empty states without CTAs — dead-end screens lose users permanently
- Ignoring network states — users expect apps to work offline at least partially
- Error messages that blame the user — "You entered an invalid email" vs "Please check your email format"

## Verification
- Trigger every error state in the app (network disconnect, invalid inputs, 404 URLs)
- Check: every error has (1) what happened, (2) why, (3) what to do next
- Test form validation: submit empty form, submit invalid data — all errors show inline with clear messages
- Verify offline behavior: disconnect network, perform actions, reconnect — data syncs correctly
- Check screen reader announces errors: `role="alert"` or `aria-live="assertive"` on error messages