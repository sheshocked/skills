---
name: secrets-management-vault
description: Deploy HashiCorp Vault, configure transit engine, manage policy access, and read secrets in production.
category: security
tags: [vault, hashicorp, secrets, api-security, production]
---

# Secrets Management Vault

## When to Use
Use when managing high-risk production variables (API tokens, private database passwords, encryption keys) to avoid exposing secrets in code commits or environment files.

## Prerequisites
- Vault binary installed on server.

## Workflow
1. Initialize and unseal the Vault instance.
2. Enable key-value (KV) engines.
3. Configure authentication policies restricting client apps scopes.
4. Integrate Vault API into application initialization sequences.

## Key Patterns

### Python Vault API Client (vault_client.py)
```python
import os
import hvac

def load_db_credentials():
    vault_addr = os.environ.get("VAULT_ADDR", "http://127.0.0.1:8200")
    vault_token = os.environ.get("VAULT_TOKEN") # Set via runtime env

    client = hvac.Client(url=vault_addr, token=vault_token)
    if not client.is_authenticated():
        raise Exception("Vault authentication failed")

    # Read database credentials secret path
    secret_response = client.secrets.kv.v2.read_secret_version(
        path="database/production",
        mount_point="secret"
    )
    
    credentials = secret_response["data"]["data"]
    return credentials["username"], credentials["password"]
```

## Pitfalls
- **Running Vault in dev mode in production:** `vault server -dev` keeps secrets in memory and turns off TLS. Always deploy in production mode with valid TLS configurations.
- **Unsealed vault lockups:** When VPS restarts, Vault locks (seals) itself. Script dynamic auto-unsealing processes using cloud KMS or secure remote keys.

## Verification
- Verify token constraints: `vault token lookup` should return restricted access scopes.
- Attempt reading path without token; verify it throws forbidden errors.

