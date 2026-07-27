---
name: appsec-code-review
description: 
category: security
tags: [appsec-code-review]
---

## When to Use
Use this skill when performing security-focused code review — taint tracking, authorization checks, deserialization vulnerabilities, dependency CVEs, and secure coding patterns.

## Core Concepts
- **Taint tracking**: Trace untrusted input from source to sink (SQL, shell, HTML)
- **Authorization checks**: Verify access control at every endpoint/function
- **Deserialization**: Unsafe deserialization leads to RCE (pickle, YAML, XML)
- **Dependency vulnerabilities**: Third-party libraries with known CVEs
- **Input validation**: Whitelist validation, parameterized queries, output encoding
- **Secure defaults**: Fail closed, deny by default, minimal privileges

## Workflow
1. **Scope**: Identify critical code paths (auth, payment, data access)
2. **Dependency scan**: Run SCA tools for known CVEs
3. **Taint analysis**: Trace user input to dangerous sinks
4. **Auth review**: Verify every endpoint has proper authorization
5. **Deserialization review**: Check all deserialization points
6. **Crypto review**: Verify proper algorithm usage and key management
7. **Error handling**: Ensure errors don't leak sensitive information
8. **Documentation**: Track findings with severity and remediation

## Key Patterns

### Taint Tracking — SQL Injection
```python
# VULNERABLE — string interpolation in SQL
def get_user(username: str):
    query = f"SELECT * FROM users WHERE username = '{username}'"  # TAINTED
    cursor.execute(query)

# FIXED — parameterized query
def get_user(username: str):
    query = "SELECT * FROM users WHERE username = %s"
    cursor.execute(query, (username,))  # Parameterized

# VULNERABLE — ORM raw query
def search(term: str):
    User.objects.raw(f"SELECT * FROM users WHERE name LIKE '%{term}%'")

# FIXED — ORM safe query
def search(term: str):
    User.objects.filter(name__icontains=term)
```

### Taint Tracking — Command Injection
```python
# VULNERABLE — subprocess with shell=True
import subprocess
def ping_host(host: str):
    subprocess.call(f"ping -c 1 {host}", shell=True)  # TAINTED

# FIXED — subprocess without shell
def ping_host(host: str):
    subprocess.call(["ping", "-c", "1", host])  # Safe

# VULNERABLE — os.system
import os
def read_file(filename: str):
    os.system(f"cat {filename}")  # TAINTED

# FIXED — use built-in file operations
def read_file(filename: str):
    with open(filename) as f:
        return f.read()
```

### Authorization Checks
```python
# VULNERABLE — no authorization check
@app.route("/api/users/<int:user_id>")
def get_user(user_id):
    user = User.query.get(user_id)  # Any user can view any user
    return jsonify(user.to_dict())

# FIXED — verify requester has access
@app.route("/api/users/<int:user_id>")
@login_required
def get_user(user_id):
    if current_user.id != user_id and not current_user.is_admin:
        abort(403)
    user = User.query.get_or_404(user_id)
    return jsonify(user.to_dict())

# VULNERABLE — IDOR via predictable IDs
@app.route("/api/invoices/<int:invoice_id>")
def get_invoice(invoice_id):
    invoice = Invoice.query.get(invoice_id)  # Sequential IDs = guessable

# FIXED — verify ownership
@app.route("/api/invoices/<int:invoice_id>")
@login_required
def get_invoice(invoice_id):
    invoice = Invoice.query.get_or_404(invoice_id)
    if invoice.user_id != current_user.id:
        abort(403)
    return jsonify(invoice.to_dict())
```

### Deserialization Vulnerabilities
```python
# VULNERABLE — pickle deserialization (RCE possible)
import pickle
def load_user_data(data: bytes):
    return pickle.loads(data)  # TAINTED — RCE

# FIXED — use JSON or safe format
import json
def load_user_data(data: str):
    return json.loads(data)

# VULNERABLE — YAML with yaml.load
import yaml
def load_config(data: str):
    return yaml.load(data)  # TAINTED — RCE via !!python/object

# FIXED — use safe_load
import yaml
def load_config(data: str):
    return yaml.safe_load(data)

# VULNERABLE — XML parsing with external entities
import xml.etree.ElementTree as ET
def parse_xml(data: str):
    return ET.fromstring(data)  # XXE possible

# FIXED — disable external entities
from defusedxml import ElementTree
def parse_xml(data: str):
    return ElementTree.fromstring(data)
```

### Dependency Scanning
```bash
# Python — safety check
pip install safety
safety check --json

# Python — pip-audit
pip install pip-audit
pip-audit --json

# JavaScript — npm audit
npm audit --json
npm audit fix

# JavaScript — Snyk
npx snyk test

# Go — govulncheck
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...

# Rust — cargo-audit
cargo install cargo-audit
cargo audit

# CI/CD integration
# .github/workflows/security.yml
name: Security Scan
on: [push, pull_request]
jobs:
  sast:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Run Trivy SAST
      uses: aquasecurity/trivy-action@master
      with:
        scan-type: 'fs'
        scan-ref: '.'
        severity: 'HIGH,CRITICAL'
        exit-code: '1'
```

### Error Handling Review
```python
# VULNERABLE — leaking stack traces
@app.errorhandler(500)
def handle_error(e):
    return jsonify({"error": str(e), "traceback": traceback.format_exc()}), 500

# FIXED — generic error messages
@app.errorhandler(500)
def handle_error(e):
    app.logger.error(f"Internal error: {e}", exc_info=True)  # Log detailed error
    return jsonify({"error": "Internal server error"}), 500  # Return generic message

# VULNERABLE — verbose error messages
def login(username, password):
    user = User.query.filter_by(username=username).first()
    if not user:
        return "User not found", 404  # Enables enumeration
    if not user.check_password(password):
        return "Wrong password", 401
    return "OK", 200

# FIXED — generic auth error
def login(username, password):
    user = User.query.filter_by(username=username).first()
    if not user or not user.check_password(password):
        return "Invalid credentials", 401  # Same message for both cases
    return "OK", 200
```

### Security-Focused Code Review Checklist
```markdown
## Authentication & Session
- [ ] Passwords hashed with bcrypt/argon2 (not MD5/SHA1)
- [ ] Session tokens have sufficient entropy (128+ bits)
- [ ] Session timeout configured (idle and absolute)
- [ ] MFA available and enforced for admin accounts
- [ ] Password policy enforced (length, complexity)

## Input Validation
- [ ] All user inputs validated on server side
- [ ] Whitelist validation (not blacklist)
- [ ] Parameterized queries for all database access
- [ ] Output encoding for HTML/JS contexts
- [ ] File upload validation (type, size, content)

## Authorization
- [ ] Every endpoint has authorization check
- [ ] Object-level authorization (not just function-level)
- [ ] Role-based access properly enforced
- [ ] API keys/tokens validated per request
- [ ] Rate limiting on authentication endpoints

## Cryptography
- [ ] No custom crypto implementations
- [ ] Proper key management (not hardcoded)
- [ ] TLS 1.2+ enforced
- [ ] Secure random for tokens (secrets module)
- [ ] IVs/nonces are unique and random
```

## Pitfalls
- **False sense of security**: Automated SAST misses logic bugs — manual review essential
- **Context matters**: A SQL query may be safe if input is from trusted source — follow the taint trail
- **Framework security**: Many frameworks have built-in protections — understand what they provide
- **Dependency depth**: Direct dependencies may have vulnerable transitive dependencies
- **Race conditions**: Check for TOCTOU (time-of-check-time-of-use) vulnerabilities
- **Logging sensitive data**: Don't log passwords, tokens, or PII — review log statements

## Verification
- All SQL queries are parameterized (no string interpolation)
- Every API endpoint has authorization check
- No deserialization of untrusted data
- Dependencies have no known HIGH/CRITICAL CVEs
- Error messages don't leak internal details
- All crypto uses established libraries
- Code review checklist completed for all critical paths