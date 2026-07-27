---
name: api-security
description: 
category: security
tags: [api-security]
---

## When to Use
Use this skill when securing APIs — OAuth2/JWT implementation, rate limiting, object-level authorization, schema validation, and abuse prevention.

## Core Concepts
- **OAuth2/OIDC**: Token-based authentication with proper grant types
- **JWT security**: Proper signing, validation, expiration, and claim verification
- **Rate limiting**: Protect against brute-force, DoS, and scraping
- **BOLA/IDOR**: Broken Object Level Authorization — most common API vulnerability
- **Schema validation**: Strict input validation with OpenAPI/JSON Schema
- **Abuse prevention**: Detecting and blocking automated attacks

## Workflow
1. **Authentication design**: Choose OAuth2 grant types based on client type
2. **Token security**: Implement proper JWT signing, validation, and rotation
3. **Authorization checks**: Verify object-level permissions on every endpoint
4. **Input validation**: Strict schema validation for all requests
5. **Rate limiting**: Implement per-user, per-endpoint rate limits
6. **API gateway**: Deploy WAF/rate limiting at gateway level
7. **Monitoring**: Track API usage patterns for anomaly detection
8. **Documentation**: Publish security requirements in API docs

## Key Patterns

### OAuth2 Implementation (Authorization Code + PKCE)
```python
# Authorization server setup (using authlib)
from authlib.integrations.flask_oauth2 import AuthorizationServer
from authlib.oauth2.rfc6749 import grants

# Client registration with PKCE
# POST /oauth/register
{
    "client_name": "MyApp",
    "redirect_uris": ["https://myapp.com/callback"],
    "grant_types": ["authorization_code"],
    "response_types": ["code"],
    "token_endpoint_auth_method": "none"  # Public client (SPA/mobile)
}

# Authorization request
# GET /oauth/authorize?
#   response_type=code&
#   client_id=myapp&
#   redirect_uri=https://myapp.com/callback&
#   code_challenge=abc123&
#   code_challenge_method=S256&
#   scope=read write

# Token exchange
# POST /oauth/token
#   grant_type=authorization_code&
#   code=AUTH_CODE&
#   redirect_uri=https://myapp.com/callback&
#   client_id=myapp&
#   code_verifier=original_challenge_string

# Token response
{
    "access_token": "eyJhbGciOiJSUzI1NiIs...",
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_token": "dGhpcyBpcyBhIHJlZnJl..."
}
```

### JWT Security Validation
```python
import jwt
from datetime import datetime, timedelta

# Token creation (with proper claims)
def create_access_token(user_id: int, scopes: list[str]) -> str:
    payload = {
        "sub": str(user_id),
        "iss": "https://auth.example.com",
        "aud": "https://api.example.com",
        "exp": datetime.utcnow() + timedelta(hours=1),
        "iat": datetime.utcnow(),
        "jti": str(uuid.uuid4()),  # Unique token ID for revocation
        "scope": " ".join(scopes)
    }
    return jwt.encode(payload, PRIVATE_KEY, algorithm="RS256")

# Token validation (comprehensive)
def validate_token(token: str) -> dict:
    try:
        payload = jwt.decode(
            token,
            PUBLIC_KEY,
            algorithms=["RS256"],  # Explicitly allow only RS256
            audience="https://api.example.com",
            issuer="https://auth.example.com",
            options={
                "require": ["exp", "iss", "sub", "aud", "iat", "jti"],
                "verify_exp": True,
                "verify_iss": True,
                "verify_aud": True,
            }
        )

        # Check if token is revoked
        if is_token_revoked(payload["jti"]):
            raise ValueError("Token revoked")

        return payload

    except jwt.ExpiredSignatureError:
        raise ValueError("Token expired")
    except jwt.InvalidAudienceError:
        raise ValueError("Invalid audience")
    except jwt.InvalidIssuerError:
        raise ValueError("Invalid issuer")
    except jwt.DecodeError:
        raise ValueError("Invalid token")
```

### Rate Limiting
```python
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

# Initialize rate limiter
limiter = Limiter(
    app=app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"],
    storage_uri="redis://localhost:6379"
)

# Per-endpoint rate limits
@app.route("/api/login", methods=["POST"])
@limiter.limit("5 per minute")  # Brute-force protection
def login():
    # ...

@app.route("/api/search", methods=["GET"])
@limiter.limit("30 per minute")  # Prevent scraping
def search():
    # ...

@app.route("/api/export", methods=["GET"])
@limiter.limit("3 per hour")  # Heavy operation
def export():
    # ...

# Custom rate limit key (per user, not per IP)
def get_user_id():
    token = request.headers.get("Authorization", "").replace("Bearer ", "")
    try:
        payload = jwt.decode(token, PUBLIC_KEY, algorithms=["RS256"])
        return payload["sub"]
    except:
        return get_remote_address()

limiter.key_func = get_user_id

# Rate limit headers (RFC 6585)
@app.after_request
def add_rate_limit_headers(response):
    if hasattr(g, 'limiter'):
        response.headers['X-RateLimit-Limit'] = g.limiter.limit
        response.headers['X-RateLimit-Remaining'] = g.limiter.remaining
        response.headers['X-RateLimit-Reset'] = g.limiter.reset
    return response
```

### Object-Level Authorization (BOLA Prevention)
```python
# VULNERABLE — no object-level auth check
@app.route("/api/orders/<int:order_id>")
def get_order(order_id):
    order = Order.query.get(order_id)  # Any user can access any order
    return jsonify(order.to_dict())

# FIXED — verify ownership
@app.route("/api/orders/<int:order_id>")
@login_required
def get_order(order_id):
    order = Order.query.get_or_404(order_id)
    if order.user_id != current_user.id:
        abort(403)  # Or 404 to avoid information leakage
    return jsonify(order.to_dict())

# Helper decorator for reusable object-level auth
from functools import wraps

def owner_required(model_class, owner_field='user_id'):
    def decorator(f):
        @wraps(f)
        @login_required
        def decorated_function(*args, **kwargs):
            obj_id = kwargs.get('id') or kwargs.get(f'{model_class.__name__.lower()}_id')
            obj = model_class.query.get_or_404(obj_id)
            if getattr(obj, owner_field) != current_user.id:
                abort(403)
            kwargs['obj'] = obj  # Pass validated object to handler
            return f(*args, **kwargs)
        return decorated_function
    return decorator

@app.route("/api/orders/<int:order_id>")
@owner_required(Order)
def get_order(order_id, obj):
    return jsonify(obj.to_dict())
```

### Schema Validation
```python
from pydantic import BaseModel, Field, validator
from typing import Optional

# Strict input schema
class CreateUserRequest(BaseModel):
    username: str = Field(..., min_length=3, max_length=32, regex=r'^[a-zA-Z0-9_]+$')
    email: str = Field(..., max_length=255)
    password: str = Field(..., min_length=12, max_length=128)

    @validator('email')
    def validate_email(cls, v):
        if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', v):
            raise ValueError('Invalid email format')
        return v.lower()

    class Config:
        # Prevent additional properties
        extra = 'forbid'

# API endpoint with schema validation
@app.route("/api/users", methods=["POST"])
def create_user():
    try:
        data = CreateUserRequest(**request.json)
    except ValidationError as e:
        return jsonify({"error": "Validation failed", "details": e.errors()}), 400

    # Now data is validated and sanitized
    user = User(username=data.username, email=data.email)
    user.set_password(data.password)
    db.session.add(user)
    db.session.commit()
    return jsonify(user.to_dict()), 201

# Response schema validation
class UserResponse(BaseModel):
    id: int
    username: str
    email: str
    created_at: datetime

    class Config:
        orm_mode = True

@app.route("/api/users/<int:user_id>")
def get_user(user_id):
    user = User.query.get_or_404(user_id)
    return jsonify(UserResponse.from_orm(user).dict())
```

### Abuse Prevention
```python
# Detect and block automated attacks

# 1. CAPTCHA after repeated failures
@app.route("/api/login", methods=["POST"])
def login():
    ip = request.remote_addr
    failures = cache.get(f"login_failures:{ip}") or 0

    if failures >= 3:
        # Require CAPTCHA
        if not verify_captcha(request.json.get("captcha_token")):
            return jsonify({"error": "CAPTCHA required"}), 400

    # Validate credentials
    user = authenticate(request.json["username"], request.json["password"])
    if not user:
        cache.set(f"login_failures:{ip}", failures + 1, timeout=900)  # 15 min
        return jsonify({"error": "Invalid credentials"}), 401

    cache.delete(f"login_failures:{ip}")
    return jsonify({"token": create_token(user)})

# 2. API key abuse detection
@app.before_request
def check_api_key_abuse():
    api_key = request.headers.get("X-API-Key")
    if api_key:
        usage = cache.get(f"api_usage:{api_key}") or {"count": 0, "window": time.time()}
        usage["count"] += 1

        if usage["count"] > 1000:  # 1000 requests per window
            return jsonify({"error": "Rate limit exceeded"}), 429

        cache.set(f"api_usage:{api_key}", usage, timeout=3600)

# 3. Input length limits (prevent memory exhaustion)
@app.before_request
def limit_content_length():
    if request.content_length and request.content_length > 10 * 1024 * 1024:  # 10MB
        return jsonify({"error": "Request too large"}), 413
```

### API Security Headers
```python
@app.after_request
def set_security_headers(response):
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['X-XSS-Protection'] = '1; mode=block'
    response.headers['Strict-Transport-Security'] = 'max-age=31536000; includeSubDomains'
    response.headers['Content-Security-Policy'] = "default-src 'self'"
    response.headers['Cache-Control'] = 'no-store, no-cache, must-revalidate'
    response.headers['Pragma'] = 'no-cache'

    # API-specific headers
    response.headers['X-API-Version'] = '1.0'
    response.headers['X-RateLimit-Policy'] = 'fixed-window'

    return response
```

## Pitfalls
- **JWT algorithm confusion**: Always verify `alg` claim — never allow `none` or `HS256` when expecting `RS256`
- **Refresh token storage**: Never store refresh tokens in localStorage — use httpOnly cookies
- **BOLA is #1**: Most common API vulnerability — check object ownership on every endpoint
- **Rate limit bypass**: Clients can rotate IPs — use API keys or JWT claims for rate limiting
- **Schema validation gaps**: Missing `extra='forbid'` allows unexpected fields through
- **CORS misconfiguration**: `Access-Control-Allow-Origin: *` on authenticated endpoints is dangerous

## Verification
- All API endpoints have proper authentication (no unauthenticated access to data)
- Object-level authorization verified on all data-access endpoints
- Rate limits active on authentication and heavy-operation endpoints
- JWT tokens validated with correct algorithm, audience, and issuer
- Input validation rejects unexpected fields and malformed data
- Security headers present on all API responses
- API documentation includes security requirements
- Penetration test confirms no BOLA, injection, or auth bypass vulnerabilities