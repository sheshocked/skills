---
name: technical-writing
description: 
category: general
tags: [technical-writing]
---

## When to Use
Write clear technical documentation: README files, API docs, tutorials, architecture decision records.

## Document Types
1. **README**: Project overview, quickstart, contribution guide
2. **Tutorial**: Step-by-step learning path
3. **Reference**: Exhaustive API/config documentation
4. **ADR**: Architecture decision records (context, decision, consequences)

## README Template
```markdown
# Project Name
> One-line description

## Features
- Feature 1
- Feature 2

## Quickstart
```bash
git clone repo
cd project
make install
```

## Usage
```code example```

## Configuration
| Variable | Default | Description |
|---|---|---|
| PORT | 8080 | Server port |

## Contributing
1. Fork
2. Branch
3. PR

## License
MIT
```

## Writing Rules
1. **Lead with why**: Explain the problem before the solution
2. **Progressive disclosure**: Simple → complex
3. **Code over prose**: Show, don't tell
4. **Concise**: One idea per sentence
5. **Active voice**: "Configure X" not "X should be configured"

## Pitfalls
- **Assumed knowledge**: Define jargon for target audience
- **Outdated docs**: Link to version-controlled docs
- **Missing examples**: Every config option needs an example
- **Wall of text**: Use headers, lists, tables to break content

## Verification
- Test quickstart on fresh machine
- Have someone follow tutorial without hints
- Check all code examples run correctly
- Verify links aren't broken