---
name: secrets-management
description: 
category: security
tags: [secrets-management]
---

## When to Use
Use this skill when implementing secrets management — HashiCorp Vault workflows, SOPS/age encryption, git-leaks scanning, environment variable hygiene, and secret rotation policies.

## Core Concepts
- **Zero plaintext**: Secrets never stored in plaintext — encrypted at rest, in transit, and in memory
- **Least privilege access**: Applications only access secrets they need
- **Rotation**: Automated secret rotation reduces blast radius of compromise
- **Audit trail**: All secret access logged with caller identity
- **Git hygiene**: No secrets in source code or git history
- **External secrets**: Pull from Vault/KMS at runtime, not embed in config

## Workflow
1. **Audit current state**: Scan repos for leaked secrets
2. **Choose vault solution**: HashiCorp Vault, AWS SSM, age+SOPS for file encryption
3. **Implement scanning**: Pre-commit hooks + CI/CD scanning
4. **Migrate secrets**: Move secrets from config files to vault
5. **Set up rotation**: Automated rotation for database passwords, API keys
6. **Access control**: Define policies for who/what can access which secrets
7. **Monitor access**: Log all secret access and review regularly

## Key Patterns

### Git Secret Scanning
```bash
# Install gitleaks
go install github.com/gitleaks/gitleaks/v8@latest

# Scan entire repo
gitleaks detect --source . --verbose --report-path gitleaks-report.json

# Scan specific commits
gitleaks detect --log-opts="--all" --report-path gitleaks-report.json

# Pre-commit hook (.pre-commit-config.yaml)
repos:
- repo: https://github.com/gitleaks/gitleaks
  rev: v8.18.0
  hooks:
  - id: gitleaks

# Scan CI/CD pipeline
# .github/workflows/security.yml
name: Secret Scanning
on: [push, pull_request]
jobs:
  gitleaks:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0
    - uses: gitleaks/gitleaks-action@v2
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### SOPS + age Encryption
```bash
# Install age encryption
apt install age
# or
go install filippo.io/age/cmd/age@latest

# Generate key pair
age-keygen -o /root/.age/key.txt
# Output: public key: age1xxxxxxxxxx

# Create .sops.yaml
cat > .sops.yaml << 'EOF'
creation_rules:
  - path_regex: secrets\.yaml$
    age: age1xxxxxxxxxx
  - path_regex: config\.enc\.json$
    age: age1xxxxxxxxxx
EOF

# Encrypt a file
sops --encrypt --age age1xxxxxxxxxx secrets.yaml > secrets.enc.yaml

# Decrypt a file
sops --decrypt secrets.enc.yaml

# Edit encrypted file in-place
sops secrets.enc.yaml

# Encrypt entire directory
find . -name "*.yaml" -exec sops --encrypt --in-place {} \;

# YAML example — secrets.enc.yaml (after encryption)
# sops:
#     kms: []
#     gcp_kms: []
#     azure_kv: []
#     age: age1xxxxxxxxxx
#     lastmodified: "2024-01-15T10:00:00Z"
#     encrypted_regex: ^(password|secret|key|token)$
#     version: 3.8.1
# db_password: ENC[AES256_GCM,data:...]
# api_key: ENC[AES256_GCM,data:...]
```

### HashiCorp Vault Workflow
```bash
# Start dev server (testing only)
vault server -dev -dev-root-token-id=root

# Enable secrets engine
vault secrets enable -path=secret kv-v2

# Store secrets
vault kv put secret/myapp/db \
  username="admin" \
  password="s3cret_p@ss"

# Read secrets
vault kv get -field=password secret/myapp/db

# Enable transit encryption (encryption as a service)
vault secrets enable transit
vault write -f transit/keys/myapp-key type=aes256-gcm96

# Encrypt data
vault write transit/encrypt/myapp-key plaintext=$(base64 <<< "sensitive data")

# Decrypt data
vault read transit/decrypt/myapp-key ciphertext="vault:v1:..."

# Dynamic database credentials
vault secrets enable database
vault write database/config/postgres \
  plugin_name=postgresql-database-plugin \
  connection_url="postgresql://{{username}}:{{password}}@db:5432/app" \
  allowed_roles="myapp" \
  username="vault_admin" \
  password="admin_pass"

vault write database/roles/myapp \
  db_name=postgres \
  default_ttl="1h" \
  max_ttl="24h"

# Get dynamic credentials
vault read database/creds/myapp
# Returns: username=v-app-myapp-abc123 password=xYz789...
```

### Pre-commit Hooks
```yaml
# .pre-commit-config.yaml
repos:
- repo: https://github.com/gitleaks/gitleaks
  rev: v8.18.0
  hooks:
  - id: gitleaks

- repo: https://github.com/antonbabenko/pre-commit-terraform
  rev: v1.83.0
  hooks:
  - id: terraform_validate
  - id: terraform_tflint

# Custom hook to detect common secrets
- repo: local
  hooks:
  - id: detect-secrets
    name: Detect Secrets
    entry: bash -c 'grep -rn "AKIA\|password\|secret_key\|private_key" --include="*.py" --include="*.js" --include="*.yaml" && exit 1 || exit 0'
    language: system
    pass_filenames: false
```

### Environment Variable Hygiene
```bash
# NEVER do this:
export DB_PASSWORD="plaintext_password"  # Visible in /proc/PID/environ
export AWS_SECRET_ACCESS_KEY="wJalrX..."

# Instead — use a secrets loader script:
#!/bin/bash
# load-secrets.sh
set -euo pipefail

# Load from Vault
export DB_PASSWORD=$(vault kv get -field=password secret/myapp/db)
export API_KEY=$(vault kv get -field=key secret/myapp/api)

# Load from SOPS-encrypted file
export $(sops --decrypt secrets.env | xargs)

# Load from AWS SSM
export DB_PASSWORD=$(aws ssm get-parameter --name "/myapp/db/password" \
  --with-decryption --query "Parameter.Value" --output text)

# Use only in current shell scope, not exported
```

## Pitfalls
- **Git history**: Deleting a file doesn't remove it from git history — use `git filter-branch` or BFG
- **Docker secrets**: `ENV` in Dockerfile exposes secrets in image layers — use Docker secrets or mounted files
- **CI/CD logs**: Secrets in build logs are visible — mask or use secret variables
- **Rotation gaps**: Old secrets remain valid until explicitly rotated — automate rotation
- **Vault unsealing**: Vault requires unsealing after restart — use auto-unseal with cloud KMS
- **Access scope**: Don't give all applications access to all secrets — scope per-service

## Verification
- `gitleaks detect --source . --verbose` — no secrets found
- All encrypted files contain `ENC[AES256_GCM` or `ENC[age` markers
- No secrets in `git log --all -p` (use BFG to clean if found)
- Vault audit log shows all secret access
- `grep -r "password\|secret\|key" --include="*.env" .` — no plaintext secrets
- Secret rotation policies defined for all critical credentials
- Pre-commit hooks active and blocking secret commits