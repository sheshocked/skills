---
name: systematic-review
description: 
category: research
tags: [systematic-review]
---

## When to Use
Conduct systematic literature reviews: PRISMA-style, inclusion criteria, extraction tables.

## PRISMA Flow
```
Identification → Screening → Eligibility → Included
  (databases)   (title/abs)   (full text)    (synthesis)
```

## Protocol Template
```markdown
# Systematic Review: {Topic}

## Research Question
Using PICO: Population, Intervention, Comparison, Outcome

## Inclusion Criteria
- Published between {year} and {year}
- Peer-reviewed
- English language
- Primary research studies

## Exclusion Criteria
- Reviews/meta-analyses
- Conference abstracts
- Non-English

## Search Strategy
Database: {list}
Search terms: {terms}
Date range: {dates}

## Data Extraction
| Study | Year | N | Method | Finding | Quality |
|---|---|---|---|---|---|
| Author | 2023 | 100 | RCT | X | High |

## Quality Assessment
- RoB 2 for RCTs
- Newcastle-Ottawa for observational
```

## Pitfalls
- **Protocol deviation**: Follow the pre-registered protocol
- **Publication bias**: Search grey literature too
- **Quality assessment**: Don't skip this step
- **Synthesis**: Consider meta-analysis if studies are comparable

## Verification
- PRISMA checklist completed
- PRISMA flow diagram included
- Protocol registered (PROSPERO)
- Two reviewers independently screened