---
name: bash-scripting
description: 
category: general
tags: [bash-scripting]
---

## When to Use
Write robust bash scripts: error handling, traps, functions, portable patterns.

## Template
```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\\n\\t'

# Trap errors
trap 'echo "Error at line $LINENO" >&2; exit 1' ERR
trap 'cleanup' EXIT

cleanup() {
    rm -f "$TMPFILE"
}

TMPFILE=$(mktemp)

# Functions
usage() {
    cat <<EOF
Usage: $(basename "$0") [OPTIONS] <file>

Options:
  -h, --help    Show this help
  -v, --verbose Verbose output
EOF
    exit 1
}

# Parse arguments
while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help) usage ;;
        -v|--verbose) VERBOSE=1; shift ;;
        -*) echo "Unknown option: $1" >&2; usage ;;
        *) FILE="$1"; shift ;;
    esac
done

[[ -z "${FILE:-}" ]] && { echo "Error: file required" >&2; usage; }
[[ -f "$FILE" ]] || { echo "Error: file not found: $FILE" >&2; exit 1; }

# Main logic
echo "Processing $FILE..."
```

## Key Patterns
```bash
# Check if command exists
command -v docker >/dev/null 2>&1 || { echo "docker required"; exit 1; }

# Array iteration
items=("one" "two" "three")
for item in "${items[@]}"; do
    echo "$item"
done

# Parallel execution
for f in *.txt; do
    process "$f" &
done
wait

# Retry loop
for i in {1..3}; do
    command && break || sleep 2
done
```

## Pitfalls
- **set -euo pipefail**: Always use at top
- **Word splitting**: Quote variables "$var"
- **Glob expansion**: Disable with set -f if needed
- **Portability**: Use bash, not sh for modern features

## Verification
- Run with shellcheck for warnings
- Test with set -x for debugging
- Check exit codes on failure
- Test with empty/missing inputs