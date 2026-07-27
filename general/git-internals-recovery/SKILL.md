---
name: git-internals-recovery
description: 
category: general
tags: [git-internals-recovery]
---

## When to Use
Recover lost work, understand git internals, debug complex git situations.

## Recovery Commands
```bash
# Find lost commit
git reflog

# Recover deleted branch
git checkout -b recovered <commit-hash>

# Recover deleted file
git checkout HEAD~1 -- path/to/file

# Find where code was removed
git log -S "removed_function_name"

# Find who changed a line
git blame file.txt

# Interactive rebase (fix history)
git rebase -i HEAD~5
```

## Internals
```
Blob    → File content (SHA1 of content)
Tree    → Directory listing (blobs + subtrees)
Commit  → Snapshot + parent refs
Tag     → Named pointer to commit
```

## Useful Aliases
```bash
git config --global alias.lg "log --oneline --graph --all"
git config --global alias.st "status -sb"
git config --global alias.unstage "reset HEAD --"
```

## Pitfalls
- **Force push**: Never force push shared branches
- **Rebase**: Don't rebase published commits
- **Large files**: Use git-lfs for binaries
- **Submodules**: Complex — prefer monorepo when possible

## Verification
- Test recovery on a throwaway branch
- Verify reflog entries
- Check .gitignore covers sensitive files