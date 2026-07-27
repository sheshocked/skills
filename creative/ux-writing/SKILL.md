---
name: ux-writing
description: 
category: creative
tags: [ux-writing]
---

## When to Use
Write interface copy: button labels, error messages, onboarding flows, tooltips, empty states, notifications, form labels, and microcopy. Use whenever text appears in a product UI.

## Core Concepts
- **Clarity over cleverness**: Users need to understand what to do, not admire your wit. "Delete account" beats "Part ways with us."
- **Voice and tone scale**: The same brand voice adapts to context. Error messages need empathy. Success messages need celebration. Navigation needs neutrality.
- **Scannability**: Users don't read — they scan. Use short sentences, clear hierarchy, and progressive disclosure.
- **Error message anatomy**: What went wrong + Why + How to fix it. Never blame the user.
- **Microcopy as product**: The difference between "Submit" and "Create my account" is the difference between feeling like a form and feeling like a product.
- **Inclusive language**: Avoid jargon, assumptions, gendered language, and ableist metaphors ("blind spot," "sanity check").

## Workflow
1. **Audit existing copy**: Screenshot every screen. Catalog all text. Identify inconsistencies, jargon, and unclear language.
2. **Define voice principles**: 3-5 rules that govern all product copy. "We're helpful, not patronizing. Clear, not cute."
3. **Create a glossary**: Standardize terms. Is it "users" or "people"? "Sign in" or "Log in"? "Cancel" or "Go back"?
4. **Write in context**: Never write copy in a doc without seeing where it lives. The screen changes the meaning.
5. **Test with users**: Read copy to 5 users. Can they explain what each element means? If not, rewrite.
6. **Build a component library**: Store approved copy patterns (empty states, errors, confirmations) in a shared doc.
7. **Review every release**: UX writing is ongoing. New features = new copy. Add it to the QA checklist.

## Key Patterns
```markdown
# Error Message Templates
Bad: "Error 404: Page not found"
Good: "This page doesn't exist. [Go back to homepage] or [search for what you need]"

Bad: "Invalid email address"
Good: "Please enter a valid email address (example: name@company.com)"

Bad: "Something went wrong"
Good: "We couldn't save your changes. Your work is saved locally — try again in a few minutes."

# Button Label Formulas
Action + Object: "Create account," "Send message," "Delete project"
First-person: "Start my free trial" (increases conversion 90% vs "Start your free trial")
Specific outcome: "Get my report" (better than "Download" or "Submit")

# Empty State Templates
1. What this page is for
2. Why it's empty (not their fault)
3. How to fix it (actionable next step)

Example:
"No projects yet"
"Projects help you organize your work. Create your first project to get started."
[Create Project →]

# Onboarding Flow Principles
- Show, don't tell: Interactive walkthroughs beat text instructions
- One action per screen: Don't overwhelm with choices
- Progress indicator: Users need to know how much is left
- Skip option: Always let power users skip tutorial
- Celebrate completion: "You're all set!" with a clear next step

# Notification Templates
Helpful: "Your report is ready to download"
Timely: "Meeting with Sarah in 15 minutes"
Actionable: "3 items need your review before Friday"
Not spammy: Never notify for things the user didn't ask for
```

## Pitfalls
- **Cutesy error messages**: "Oops! Something went sideways 😅" doesn't help the user fix the problem
- **Blaming the user**: "You entered an invalid email" → "Please enter a valid email"
- **Jargon**: "Initiate synchronization protocol" → "Syncing your data now"
- **Inconsistent terminology**: Using "account" in one place and "profile" in another confuses users
- **Writing in isolation**: UX copy without the design context is like writing dialogue without the scene
- **Ignoring internationalization**: Text that's short in English may be long in German. Design for expansion.

## Verification
- Read all copy aloud: does it sound like a human talking to another human?
- User test: show screens to 5 people unfamiliar with the product. Can they navigate based on copy alone?
- Glossary consistency: all instances of the same concept use the same word
- Error coverage: every possible error state has a helpful, specific message
- Accessibility: copy is clear at 200% zoom and works with screen readers
- Internationalization: text is translatable without breaking layout