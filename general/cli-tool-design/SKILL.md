---
name: cli-tool-design
description: 
category: general
tags: [cli-tool-design]
---

## When to Use
Design ergonomic CLI tools: command structure, flags, help text, output formatting, shell completions.

## Principles
1. **Convention over configuration**: Sensible defaults
2. **Discoverable**: --help should answer most questions
3. **Composable**: Pipe-friendly output (JSON/text)
4. **Idempotent**: Running twice produces same result

## Command Structure
```bash
tool <command> [subcommand] [flags] [args]

# Examples
tool deploy --env staging
tool logs --tail 100 --filter "error"
tool config set api.url https://api.example.com
```

## Flag Design
```bash
# Short + long flags
-v, --verbose        # Boolean flags
-p, --port PORT      # Value flags with name
--timeout 30s        # Units in value
--no-color           # Negation prefix
```

## Output Formatting
```bash
# Default: human-readable table
tool list

# JSON for scripting
tool list --output json

# Quiet mode (just values)
tool get name --quiet
```

## Pitfalls
- **Flag conflicts**: Don't reuse short flags across commands
- **Help text**: Keep under 80 chars per line
- **Exit codes**: 0=success, 1=general error, 2=usage error
- **Stderr vs stdout**: Errors to stderr, data to stdout

## Verification
- Test with pipe: tool | jq .
- Verify help text is accurate
- Check shell completions work
- Test with invalid flags for error messages