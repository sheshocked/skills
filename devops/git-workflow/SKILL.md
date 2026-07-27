---
name: git-workflow
description: 
category: devops
tags: [git-workflow]
---

## When to Use
Implement Git branching strategies, commit conventions, release management, and collaboration workflows. Covers Git Flow, Trunk-Based Development, conventional commits, and monorepo strategies.

## Core Concepts
- **Trunk-Based Development**: Short-lived feature branches, frequent main merges
- **Git Flow**: Feature/develop/release/hotfix branches for scheduled releases
- **Conventional Commits**: `feat:`, `fix:`, `chore:`, `docs:` prefixes
- **Signed commits**: GPG/SSH signing for verified authorship
- **Monorepo**: Multiple projects in one repo with workspaces
- **Bisect**: Binary search through commits to find bugs

## Workflow
1. Choose branching strategy based on release cadence
2. Enforce commit message conventions
3. Use signed commits for security
4. Tag releases with semantic versioning
5. Automate changelogs from commit history

## Key Patterns
```bash
# Conventional commit examples
git commit -m "feat(auth): add OAuth2 login with Google"
git commit -m "fix(api): handle null response from payment provider"
git commit -m "chore(deps): bump axios from 1.6.0 to 1.6.2"
git commit -m "docs(readme): update installation instructions"
git commit -m "feat(db)!: rename user table columns"  # breaking change

# Semantic versioning with tags
git tag -a v1.2.3 -m "Release v1.2.3: OAuth2 support, bug fixes"
git push origin v1.2.3
```

```bash
# Trunk-Based Development workflow
git checkout main
git pull origin main
git checkout -b feat/my-feature
# ... work ...
git add -p  # stage interactively
git commit -m "feat(api): add endpoint for user preferences"
git fetch origin main && git rebase origin/main
git push origin feat/my-feature
# Create PR → merge to main → delete branch
git branch -d feat/my-feature
git push origin --delete feat/my-feature
```

```bash
# Signed commits — setup
git config --global user.signingkey YOUR_GPG_KEY_ID
git config --global commit.gpgsign true
git config --global tag.gpgsign true

# Verify signatures
git log --show-signature -1

# Git hooks for commit validation
# .husky/commit-msg
npx --no -- commitlint --edit $1

# commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [2, 'always', [
      'feat', 'fix', 'docs', 'style', 'refactor',
      'perf', 'test', 'chore', 'ci', 'revert'
    ]],
    'subject-max-length': [2, 'always', 72],
    'body-max-line-length': [1, 'always', 100],
  },
};
```

```bash
# Git bisect for bug hunting
git bisect start
git bisect bad HEAD
git bisect good v1.2.0
# Git checks out middle commit; test it
git bisect good  # if commit is fine
# or
git bisect bad   # if commit is broken
# Repeat until found
git bisect reset

# Interactive rebase to clean up before merge
git rebase -i HEAD~5
# pick → keep, squash → combine, reword → edit message, drop → remove
```

```bash
# Changelog generation
git log v1.2.0..v1.3.0 --pretty=format:"- %s (%h)" --no-merges

# Automated with conventional-changelog
npx conventional-changelog -p angular -i CHANGELOG.md -s
```

## Pitfalls
- **Merge conflicts**: Rebase feature branches frequently against main
- **Force push safety**: Never force push to main/master
- **Large PRs**: Keep PRs under 400 lines for effective review
- **Stale branches**: Delete merged branches immediately
- **Commit message consistency**: Enforce with commitlint in CI
- **Binary files**: Avoid committing large binaries; use Git LFS

## Verification
```bash
# Verify commit conventions
npx commitlint --from HEAD~1

# Check branch hygiene
git branch -a | grep -v main | grep -v HEAD

# Verify GPG signatures
git log --show-signature -5

# Check for untracked large files
git ls-files --others --exclude-standard | xargs -I {} sh -c 'wc -c "{}" | sort -rn' | head
```