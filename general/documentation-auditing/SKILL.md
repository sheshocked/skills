---
name: documentation-auditing
description: 
category: general
tags: [documentation-auditing]
---

## When to Use
Audit documentation: check freshness, find broken links, test examples, identify gaps.

## Audit Checklist
1. **Freshness**: When was each doc last updated?
2. **Links**: Are all internal/external links working?
3. **Examples**: Do code examples still run?
4. **Gaps**: Are important topics undocumented?
5. **Accuracy**: Does the doc match current behavior?

## Automated Checks
```bash
# Find broken links
markdown-link-check README.md

# Check code examples run
doctest --testdoctest

# Find outdated docs
find docs/ -mtime +90 -name "*.md"

# Spell check
cspell "docs/**/*.md"
```

## Pitfalls
- **Stale docs worse than no docs**: Remove outdated content
- **Version drift**: Docs may not match deployed version
- **Missing context**: Assume nothing about reader knowledge

## Verification
- Fresh eyes review for clarity
- All examples run without errors
- No broken links