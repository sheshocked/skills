---
name: search-replace-refactors
description: 
category: general
tags: [search-replace-refactors]
---

## When to Use
Large-scale code refactoring: codemods, ast-grep, find-replace across repos.

## ast-grep
```bash
# Install
npm install -g @ast-grep/cli

# Find pattern
sg run -p 'console.log($MSG)' -l js

# Replace
sg run -p 'oldFunction($ARG)' -r 'newFunction($ARG)' -l js
```

## Codemod (JS/TS)
```bash
# jscodeshift
npx jscodeshift -t transforms/rename.js src/
```

## Python (LibCST)
```python
import libcst as cst

class RenameFunc(cst.CSTTransformer):
    def leave_Call(self, original_node, updated_node):
        if isinstance(updated_node.func, cst.Name) and updated_node.func.value == "old_func":
            return updated_node.with_changes(func=cst.Name("new_func"))
        return updated_node
```

## Pitfalls
- **Always test on branch**: Never refactor on main directly
- **Regex vs AST**: AST-based is safer than regex for code
- **Review diffs**: Manual review after automated changes
- **Incremental**: Small commits for large refactors

## Verification
- Run tests after refactoring
- Check for syntax errors
- Verify no behavioral changes