---
name: ci-cd-github-actions
description: 
category: devops
tags: [ci-cd-github-actions]
---

## When to Use
Build CI/CD pipelines with GitHub Actions for automated testing, building, deploying, and releasing software. Covers workflows, reusable actions, matrix builds, caching, secrets, and deployment strategies.

## Core Concepts
- **Workflows**: YAML files in `.github/workflows/` triggered by events
- **Jobs**: Parallel or sequential units of work (run on separate runners)
- **Steps**: Individual tasks within a job (uses, run)
- **Actions**: Reusable units (marketplace or custom)
- **Matrix**: Test across multiple OS/language versions simultaneously
- **Environments**: Deploy protection rules with required reviewers

## Workflow
1. Define trigger events (push, PR, schedule, workflow_dispatch)
2. Set up matrix for multi-version testing
3. Cache dependencies for faster runs
4. Build and push container images
5. Deploy to staging automatically, production with approval
6. Create GitHub releases on tags

## Key Patterns
```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline
on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm test
      - run: npm run lint

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/tags/v')
    permissions:
      contents: read
      packages: write
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=sha
      - uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    needs: build-and-push
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy to staging
        run: |
          kubectl set image deployment/api api=${{ needs.build-and-push.outputs.image-tag }} -n staging
          kubectl rollout status deployment/api -n staging --timeout=300s

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')
    environment:
      name: production
      url: https://api.example.com
    steps:
      - name: Deploy to production
        run: |
          kubectl set image deployment/api api=${{ needs.build-and-push.outputs.image-tag }} -n production
          kubectl rollout status deployment/api -n production --timeout=300s
```

```yaml
# Reusable workflow for secret scanning
# .github/workflows/security.yml
name: Security Scan
on:
  workflow_call:
    inputs:
      image:
        required: true
        type: string
    secrets:
      SNYK_TOKEN:
        required: true

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ inputs.image }}
          format: sarif
          output: trivy-results.sarif
          severity: CRITICAL,HIGH
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: trivy-results.sarif
```

## Pitfalls
- **Secrets in logs**: Use `::add-mask::` or `>> $GITHUB_OUTPUT` carefully
- **Cache invalidation**: npm/yarn caches need correct lock file path
- **Concurrency**: Use `concurrency` groups to prevent duplicate deploys
- **Fork PRs**: Don't pass secrets to fork workflows (security risk)
- **Large artifacts**: Use `actions/cache` not `upload-artifact` for large files
- **Runner costs**: Self-hosted runners for private repos save costs

## Verification
```bash
# Validate workflow syntax
act -l  # list workflows with nektos/act

# Check workflow runs
gh run list --limit 10
gh run view <run-id>

# Verify secrets are set
gh secret list

# Test locally with act
act push -j test --secret-file .env
```