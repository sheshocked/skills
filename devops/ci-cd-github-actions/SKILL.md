---
name: ci-cd-github-actions
description: Establish build pipelines, caching dependencies, and automatic ssh deployment in GitHub actions.
category: devops
tags: [github-actions, ci-cd, caching, deploy, yaml]
---

# Ci Cd Github Actions

## When to Use
Use to automate code checks, linting, packaging, and automatic deployments.

## Prerequisites
- Project repository on GitHub.

## Workflow
1. Declare runner platforms.
2. Cache dependencies to speed up pipelines.
3. Configure secure SSH keys keys mapping deployment targets.

## Key Patterns
```yaml
name: Deploy Application
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy via SSH
        uses: appleboy/ssh-action@master
        with:
          host: 185.71.219.72
          username: root
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /root/app
            git pull
            docker compose down
            docker compose up -d --build
```

## Pitfalls
- **Configuration Cache misses:** Failing to include hashes in keys pulls old dependencies.
- **Exposing Secrets:** Never echo secret variables in standard workflow run commands.

## Verification
- Verify runner succeeds in GitHub Actions workspace.
- Check target server deployment changes.
