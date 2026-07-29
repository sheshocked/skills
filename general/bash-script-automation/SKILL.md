---
name: bash-script-automation
description: Establish strict mode scripts, process traps, cleanup functions, and log styling.
category: general
tags: [bash, scripting, automation, clean-code, linux]
---

# Bash Script Automation

## When to Use
Use to build durable installer bash scripts (e.g. RealityGhost.sh) that must run unattended on client nodes without hanging.

## Prerequisites
- Bash shell.

## Workflow
1. Enable strict flag declarations (`set -euo pipefail`).
2. Register exit trap cleanup functions.
3. Style console logs with distinct colors.

## Key Patterns
```bash
#!/bin/bash
# Strict mode
set -euo pipefail

# Colors
RED='[0;31m'
GREEN='[0;32m'
NC='[0m' # No Color

# Cleanup trap
cleanup() {
    echo -e "${RED}Script interrupted. Performing cleanup...${NC}"
    rm -f /tmp/temp_download_*
}
trap cleanup SIGINT SIGTERM EXIT

echo -e "${GREEN}Starting installation...${NC}"
# Commands here

# Clear trap on success
trap - EXIT
echo -e "${GREEN}Done!${NC}"
```

## Pitfalls
- **Failing to pipe logs safely:** Standard commands can output long traces. Redirect outputs to log files while showing progress bars to keep CLI clean.
- **Pipefail surprises:** Some common commands return non-zero exit codes on normal output. Handle them using `|| true`.

## Verification
- Test script termination (`Ctrl+C`); verify cleanup actions execute.
- Run script in dry-run mode to confirm parameter assignments.
