---
name: auth-systems
description: - Building authentication for user-facing applications
category: engineering
tags: [auth-systems]
---

## When to Use

- Building authentication for user-facing applications
- Implementing API authorization (who can access what)
- Integrating with third-party identity providers (Google, Okta, Auth0)
- Designing multi-tenant authorization (org-level, role-based, resource-based)

## Core Concepts

- **Authentication vs Authorization**: AuthN = "who are you?" AuthZ = "what can you do?" Handle them separately.
- **JWT (JSON Web Tokens)**: Stateless tokens containing claims. No server-side session storage. Verify signature, don't trust payload.
- **OAuth 2.0 Flows**: Authorization Code (web apps), Client Credentials (service-to-service), PKCE (mobile/SPA).
- **RBAC vs ABAC**: Role-Based (admin/user) vs Attribute-Based (department, clearance level, resource ownership).
- **Password Hashing**: bcrypt/scrypt/argon2 with per-user salt. NEVER MD5/SHA for passwords.

## Workflow

1. **Choose auth method** — session-based (server-side) vs token-based (JWT) vs OAuth/OIDC
2. **Implement password storage** — bcrypt with cost factor 12+, argon2 for new systems
3. **Design token lifecycle** — access token (15min) + refresh token (7 days)
4. **Implement authorization** — RBAC or ABAC based on domain complexity
5. **Add rate limiting** — auth endpoints are brute-force targets
6. **Audit logging** — log every auth event (login, logout, permission denied)

## Key Patterns

```python
# Password hashing with bcrypt
import bcrypt

def hash_password(password: str) -> str: