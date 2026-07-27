---
name: arxiv-literature-search
description: 
category: research
tags: [arxiv-literature-search]
---

## When to Use
Systematically search and synthesize academic literature from arXiv and other sources.

## Search Strategy
```bash
# arXiv API
curl "http://export.arxiv.org/api/query?search_query=cat:cs.AI+AND+all:transformers&max_results=10"

# Semantic Scholar API
curl "https://api.semanticscholar.org/graph/v1/paper/search?query=attention+is+all+you+need&limit=10"

# Connected Papers (visual exploration)
# https://www.connectedpapers.com/
```

## Synthesis Template
```markdown
# Literature Review: {Topic}

## Key Papers
1. [Author, Year] - Title
   - Claim: ...
   - Method: ...
   - Finding: ...
   - Limitation: ...

## Common Themes
- Theme 1: ...
- Theme 2: ...

## Gaps
- Gap 1: ...
- Gap 2: ...

## Research Questions
1. ...
2. ...
```

## Pitfalls
- **Recency bias**: Don't ignore older foundational work
- **Confirmation bias**: Seek disconfirming evidence
- **Scope creep**: Define search boundaries early
- **Citation chains**: Follow references both forward and backward

## Verification
- Coverage: Did you miss key papers?
- Recency: Are you citing current work?
- Balance: Are different perspectives represented?