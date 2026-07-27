---
name: regex-mastery
description: 
category: general
tags: [regex-mastery]
---

## When to Use
Write robust regex patterns: lookarounds, atomic groups, backtracking control, debugging.

## Pattern Reference
```
.         Any character
\\d        Digit [0-9]
\\w        Word character [a-zA-Z0-9_]
\\s        Whitespace
[abc]     Character set
[^abc]    Negated set
(a|b)     Alternation
a?        Optional (0 or 1)
a*        Zero or more
a+        One or more
a{3}      Exactly 3
a{3,5}    Between 3 and 5
^         Start of string
$         End of string
```

## Lookarounds
```
(?=foo)   Positive lookahead — followed by foo
(?!foo)   Negative lookahead — NOT followed by foo
(?<=foo)  Positive lookbehind — preceded by foo
(?<!foo)  Negative lookbehind — NOT preceded by foo
```

## Common Patterns
```python
import re

# Email (simplified)
r'[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}'

# IPv4
r'\\b(?:\\d{1,3}\\.){3}\\d{1,3}\\b'

# URL
r'https?://[^\\s]+'

# Date YYYY-MM-DD
r'\\d{4}-(?:0[1-9]|1[0-2])-(?:0[1-9]|[12]\\d|3[01])'
```

## Pitfalls
- **Catastrophic backtracking**: Avoid nested quantifiers (a+)+ 
- **Greedy vs lazy**: Use .*? for lazy matching
- **Unicode**: Use re.UNICODE flag for international text
- **Escaping**: \\ before special characters

## Debugging Tools
- regex101.com — visual explanation
- regexper.com — railroad diagrams
- re.compile(pattern) — Python compile check

## Verification
- Test with edge cases (empty, very long, special chars)
- Benchmark on large inputs
- Verify with multiple test cases