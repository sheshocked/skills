---
name: code-review-craft
description: - Reviewing pull requests before merge
category: engineering
tags: [code-review-craft]
---

## When to Use

- Reviewing pull requests before merge
- Establishing team review standards and checklists
- Mentoring junior engineers through review feedback
- Reducing bugs reaching production via systematic review

## Core Concepts

- **Review Pyramid**: From bottom to top — correctness → design → readability → style. Style matters least; correctness matters most.
- **Nits vs Issues**: Clearly distinguish blocking issues ("this will crash") from suggestions ("consider renaming"). Use labels.
- **Context Loading**: Read the issue/ticket first, then the PR description, then the diff. Context changes what you look for.
- **Review Time Budget**: Spend 30-60 minutes per review max. Longer reviews lose accuracy. Ask for smaller PRs.
- **Positive Feedback**: Acknowledge good patterns. "Nice use of X" reinforces behavior. Reviews aren't just about catching problems.

## Workflow

1. **Read the PR description** — what is being changed and why
2. **Check the test coverage** — do tests cover the change?
3. **Review the design** — does the approach make sense at architecture level?
4. **Review implementation** — correctness, edge cases, error handling
5. **Review readability** — naming, comments, structure
6. **Leave actionable feedback** — specific, constructive, with suggestions

## Key Patterns

```python
# Code review checklist as Python validation
REVIEW_CHECKLIST = {
    "correctness": [
        "Does the code do what the PR description says?",
        "Are edge cases handled (empty input, null, overflow)?",
        "Are error paths covered (try/except, None checks)?",
        "Is the change idempotent where needed?",
    ],
    "testing": [
        "Are there tests for new functionality?",
        "Do tests cover the happy path AND error paths?",
        "Are tests readable (arrange-act-assert)?",
        "Do tests actually assert something meaningful?",
    ],
    "design": [
        "Is the change in the right module/service?",
        "Does it follow existing patterns in the codebase?",
        "Is the API contract backward-compatible?",
        "Could this be simpler? (YAGNI check)",
    ],
    "security": [
        "Is user input validated/sanitized?",
        "Are secrets excluded from code/config?",
        "Are SQL queries parameterized (no string interpolation)?",
        "Is authorization checked before data access?",
    ],
    "performance": [
        "Are database queries indexed (check EXPLAIN)?",
        "Is N+1 fetching avoided?",
        "Are large datasets paginated?",
        "Is there unnecessary memory allocation?",
    ],
}
```

```python
# Review feedback templates (PR comment format)
REVIEW_TEMPLATES = {
    "blocking":