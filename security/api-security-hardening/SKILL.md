---
name: api-security-hardening
description: Enforce strict JWT claims validation, rate limiting, secure headers, CORS, and request validations.
category: security
tags: [api-security, jwt, cors, rate-limit, security]
---

# Api Security Hardening

## When to Use
Use when developing backend APIs that handle transaction requests or profile modifications.

## Prerequisites
- Web framework (FastAPI, Express, NestJS).

## Workflow
1. Check claims, signature, and expiration of incoming JWT tokens.
2. Inject rate limiting headers.
3. Verify request payloads strictly against schemas.

## Key Patterns
```python
# FastAPI JWT validation dependency
from fastapi import HTTPException, Security, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
import jwt

security = HTTPBearer()
JWT_SECRET = "mysecretkey"
JWT_ALGORITHM = "HS256"

def validate_token(credentials: HTTPAuthorizationCredentials = Security(security)):
    token = credentials.credentials
    try:
        payload = jwt.decode(token, JWT_SECRET, algorithms=[JWT_ALGORITHM])
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Token expired")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid token")
```

## Pitfalls
- **Allowing wildcard CORS:** Never deploy `Access-Control-Allow-Origin: *` in production APIs handling cookies. Specifiy exact domains.
- **JWT encryption limits:** Avoid storing sensitive user information inside JWT payload; keep only ID/Role keys.

## Verification
- Verify invalid or expired tokens return `401 Unauthorized`.
- Check CORS settings using dynamic preflight OPTIONS requests.
