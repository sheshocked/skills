---
name: secrets-in-ci
description: 
category: devops
tags: [secrets-in-ci]
---

## When to Use
Manage secrets, API keys, and credentials securely in CI/CD pipelines. Covers GitHub Actions secrets, Vault integration, OIDC for cloud auth, environment protection, and secret scanning.

## Core Concepts
- **CI/CD secrets**: Encrypted variables available only during workflow runs
- **OIDC**: Federated identity — no long-lived credentials needed
- **Vault**: Centralized secrets management with dynamic credentials
- **Secret scanning**: Detect leaked credentials in code/repo
- **Environment protection**: Required reviewers for production secrets
- **Masking**: Hide secrets from logs with `::add-mask::`

## Workflow
1. Store secrets at appropriate scope (repo, environment, org)
2. Use OIDC for cloud provider auth (no static keys)
3. Implement secret scanning in pre-commit and CI
4. Rotate secrets on a schedule
5. Audit secret access with logs

## Key Patterns
```yaml
# GitHub Actions — OIDC for AWS (no static keys)
name: Deploy with OIDC
on:
  push:
    branches: [main]

permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789:role/github-actions-deploy
          aws-region: us-east-1

      - name: Deploy
        run: |
          aws s3 sync ./dist s3://my-bucket/
```

```yaml
# GitHub Actions — Vault integration
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Import Secrets
        uses: hashicorp/vault-action@v3
        with:
          url: https://vault.example.com
          method: jwt
          role: github-actions
          secrets: |
            secret/data/deployment AWS_ACCESS_KEY_ID | AWS_ACCESS_KEY_ID ;
            secret/data/deployment AWS_SECRET_ACCESS_KEY | AWS_SECRET_ACCESS_KEY ;
            secret/data/database DB_PASSWORD | DATABASE_URL

      - name: Use secrets
        run: |
          echo "Deploying with credentials..."
          # Secrets available as env vars
```

```yaml
# GitHub Actions — Secret masking and protection
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # requires approval
    steps:
      - name: Mask secrets
        run: |
          echo "::add-mask::${{ secrets.API_KEY }}"
          echo "::add-mask::${{ secrets.DB_PASSWORD }}"

      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}
        run: ./deploy.sh
        # Secrets are automatically masked in logs
```

```bash
# Pre-commit secret scanning
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.1
    hooks:
      - id: gitleaks

# Manual scan
gitleaks detect --source . --verbose

# GitHub secret scanning
gh secret list
gh secret set API_KEY --body "value" --env production
gh secret delete OLD_KEY
```

```yaml
# Vault dynamic database credentials
# vault write database/roles/myapp \
#   db_name=myapp \
#   default_ttl="1h" \
#   max_ttl="24h"

# In CI — get temporary credentials
# vault read database/creds/myapp
# key        value
# ---        -----
# lease_id   database/creds/myapp/abc123
# username   myapp-user-abc123
# password   A1b2C3d4E5f6
```

```yaml
# OIDC for GCP (Workload Identity Federation)
- uses: google-github-actions/auth@v2
  with:
    workload_identity_provider: projects/123/locations/global/workloadIdentityPools/github-pool/providers/github-provider
    service_account: github-actions@myproject.iam.gserviceaccount.com
```

## Pitfalls
- **Secrets in logs**: Never `echo $SECRET` — use `::add-mask::` or avoid printing
- **Fork workflows**: Don't pass secrets to fork PRs (security risk)
- **OIDC trust**: Configure repository/branch filters on OIDC providers
- **Vault token expiry**: Use short-lived tokens; don't hardcode Vault tokens
- **Secret rotation**: Implement automated rotation for all long-lived secrets
- **Environment protection**: Require approvals for production secret access

## Verification
```bash
# Verify secrets are set
gh secret list --env production

# Test OIDC authentication
aws sts get-caller-identity  # after OIDC auth

# Scan for leaked secrets
gitleaks detect --source . --verbose
trufflehog filesystem --directory .

# Verify Vault access
vault read database/creds/myapp
```