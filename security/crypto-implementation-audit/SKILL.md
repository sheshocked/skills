---
name: crypto-implementation-audit
description: 
category: security
tags: [crypto-implementation-audit]
---

## When to Use
Use this skill when auditing cryptographic implementations for vulnerabilities — padding oracle attacks, nonce reuse, weak random number generation, side-channel leakage, and protocol-level flaws.

## Core Concepts
- **Padding oracle**: CBC mode decryption leaks valid/invalid padding via timing or error messages
- **Nonce reuse**: Reusing nonces in AES-GCM/CTR destroys confidentiality completely
- **Weak RNG**: Predictable PRNG output compromises all dependent cryptographic operations
- **Side channels**: Timing, power analysis, cache attacks leak key material
- **Protocol flaws**: Improper key derivation, missing authentication, replay attacks

## Workflow
1. **Identify crypto primitives**: AES, RSA, ChaCha20, ECDH — map the crypto surface
2. **Check nonce/IV handling**: Verify uniqueness, randomness, and proper size
3. **Test for padding oracle**: Modify ciphertext, observe error differences
4. **Audit RNG**: Check seed entropy, predictability, reseeding
5. **Review key management**: Storage, derivation (KDF parameters), rotation
6. **Check authentication**: MAC verification, AEAD nonce handling
7. **Side-channel review**: Timing consistency, constant-time operations
8. **Protocol analysis**: Replay protection, forward secrecy, session management

## Key Patterns

### Padding Oracle Detection
```python
#!/usr/bin/env python3
# Detect CBC padding oracle vulnerability
import requests
import sys
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes

def test_padding_oracle(url, token_field='token'):
    # Test for padding oracle by modifying ciphertext bytes
    # Get a valid token
    resp = requests.get(f"{url}/login?user=admin")
    valid_token = resp.cookies.get('session')

    # Modify last byte of ciphertext
    modified = bytearray(bytes.fromhex(valid_token))
    modified[-1] ^= 0x01  # flip last byte
    modified_hex = modified.hex()

    resp_modified = requests.get(
        f"{url}/dashboard",
        cookies={'session': modified_hex}
    )

    # Check for different error responses
    resp_invalid = requests.get(
        f"{url}/dashboard",
        cookies={'session': '00' * 32}
    )

    if resp_modified.status_code != resp_invalid.status_code:
        print("[VULNERABLE] Padding oracle detected — different responses")
        return True

    # Also check timing
    import time
    times = []
    for _ in range(10):
        start = time.time()
        requests.get(f"{url}/dashboard", cookies={'session': modified_hex})
        times.append(time.time() - start)

    avg_time = sum(times) / len(times)

    times_invalid = []
    for _ in range(10):
        start = time.time()
        requests.get(f"{url}/dashboard", cookies={'session': '00' * 32})
        times_invalid.append(time.time() - start)

    avg_time_invalid = sum(times_invalid) / len(times_invalid)

    if abs(avg_time - avg_time_invalid) > 0.01:  # 10ms difference
        print("[VULNERABLE] Padding oracle via timing side-channel")
        return True

    return False

if __name__ == "__main__":
    test_padding_oracle(sys.argv[1])
```

### Nonce Reuse Detection
```python
#!/usr/bin/env python3
# Detect AES-GCM nonce reuse in application traffic
import hashlib
from collections import defaultdict

def detect_nonce_reuse(ciphertexts: list[bytes]) -> bool:
    # Check if any AES-GCM nonces are reused
    seen_nonces = defaultdict(int)

    for ct in ciphertexts:
        # AES-GCM: nonce is typically first 12 bytes of ciphertext
        # Or appended as last 12 bytes (varies by implementation)
        nonce = ct[:12]
        seen_nonces[nonce] += 1

    for nonce, count in seen_nonces.items():
        if count > 1:
            print(f"[CRITICAL] Nonce reuse detected: {nonce.hex()} used {count} times")
            return True

    print("[OK] No nonce reuse detected")
    return False

def test_aes_gcm_nonce_misuse():
    # Demonstrate catastrophic failure from nonce reuse
    from cryptography.hazmat.primitives.ciphers.aead import AESGCM
    import os

    key = AESGCM.generate_key(bit_length=256)
    nonce = os.urandom(12)  # FIXED nonce — dangerous!

    aesgcm = AESGCM(key)

    # Encrypt two messages with same nonce
    ct1 = aesgcm.encrypt(nonce, b"secret_message_1", None)
    ct2 = aesgcm.encrypt(nonce, b"secret_message_2", None)

    # XOR of plaintexts = XOR of ciphertexts (tag reveals relationship)
    # This is a catastrophic break
    plaintexts_xor = bytes(a ^ b for a, b in zip(ct1[12:], ct2[12:]))
    print(f"XOR of plaintexts: {plaintexts_xor}")
```

### Weak RNG Audit
```python
#!/usr/bin/env python3
# Audit random number generation in applications
import random
import hashlib
import time
from collections import Counter

def test_python_random():
    # Python's random module is NOT cryptographically secure
    # This is VULNERABLE — seed is time-based
    random.seed(int(time.time()))
    token = random.getrandbits(128)
    print(f"[INSECURE] random.getrandbits: {token:#x}")
    print(f"[INSECURE] Predictable if seed time is known")

def test_secrets_module():
    # Use secrets module for cryptographic operations
    import secrets
    token = secrets.token_bytes(32)
    print(f"[SECURE] secrets.token_bytes: {token.hex()}")

def test_openssl_random():
    # Verify OpenSSL CSPRNG availability
    import os
    random_bytes = os.urandom(32)
    print(f"[SECURE] os.urandom: {random_bytes.hex()}")

def detect_weak_prng(values: list[int]) -> bool:
    # Check if generated values show detectable patterns
    # Check for low entropy
    bits = ''.join(format(v, '032b') for v in values[:100])
    ones = bits.count('1')
    zeros = bits.count('0')
    ratio = min(ones, zeros) / max(ones, zeros)

    if ratio < 0.45:
        print(f"[WEAK] Bit distribution suspicious: {ratio:.3f}")
        return True

    # Check for sequential/predictable patterns
    diffs = [values[i+1] - values[i] for i in range(len(values)-1)]
    if len(set(diffs)) < 5:
        print(f"[WEAK] Only {len(set(diffs))} unique differences — predictable")
        return True

    return False
```

### Timing Side-Channel Test
```python
#!/usr/bin/env python3
# Test for timing side-channels in crypto operations
import time
import statistics

def measure_comparison_timing(target_func, correct_input, wrong_input, iterations=10000):
    # Measure if comparison timing leaks information
    times_correct = []
    times_wrong = []

    for _ in range(iterations):
        start = time.perf_counter_ns()
        target_func(correct_input)
        times_correct.append(time.perf_counter_ns() - start)

        start = time.perf_counter_ns()
        target_func(wrong_input)
        times_wrong.append(time.perf_counter_ns() - start)

    median_correct = statistics.median(times_correct)
    median_wrong = statistics.median(times_wrong)
    diff = abs(median_correct - median_wrong)

    print(f"Correct input median: {median_correct}ns")
    print(f"Wrong input median:   {median_wrong}ns")
    print(f"Difference:           {diff}ns")

    if diff > 100:  # >100ns difference is suspicious
        print("[VULNERABLE] Timing side-channel detected")
        return True

    print("[OK] No timing side-channel detected")
    return False

def constant_time_compare(a: bytes, b: bytes) -> bool:
    # Constant-time comparison (correct implementation)
    import hmac
    return hmac.compare_digest(a, b)

def vulnerable_compare(a: bytes, b: bytes) -> bool:
    # VULNERABLE: early-exit comparison
    if len(a) != len(b):
        return False
    for x, y in zip(a, b):
        if x != y:
            return False  # Early exit leaks position
    return True
```

### Key Derivation Audit
```python
#!/usr/bin/env python3
# Audit key derivation functions
import hashlib
import bcrypt
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes

def audit_kdf(password: str, salt: bytes) -> bool:
    # Check KDF parameters for security
    issues = []

    # Check PBKDF2 iterations (should be >= 600,000 for SHA-256)
    kdf = PBKDF2HMAC(
        algorithm=hashes.SHA256(),
        length=32,
        salt=salt,
        iterations=100_000,  # TOO LOW
    )
    key = kdf.derive(password.encode())

    if kdf.iterations < 600_000:
        issues.append(f"Iterations too low: {kdf.iterations} (should be >= 600,000)")

    # Check if scrypt/argon2 should be used instead
    # PBKDF2 is acceptable but Argon2id is preferred
    issues.append("Consider Argon2id for password hashing (memory-hard)")

    # Check salt randomness
    if salt == b'\x00' * len(salt):
        issues.append("Salt is all zeros — not random")

    for issue in issues:
        print(f"[ISSUE] {issue}")

    return len(issues) == 0

def secure_kdf(password: str) -> bytes:
    # Secure key derivation with Argon2id
    import argon2
    # Argon2id is preferred for password hashing
    kdf = argon2.low_level.hash_secret_raw(
        secret=password.encode(),
        salt=argon2.low_level.rand_bytes(16),
        time_cost=3,        # iterations
        memory_cost=65536,  # 64MB
        parallelism=4,
        hash_len=32,
        type=argon2.low_level.Type.ID
    )
    return kdf
```

## Pitfalls
- **Never roll your own crypto**: Use established libraries (cryptography, OpenSSL, libsodium)
- **Nonce uniqueness**: AES-GCM nonce must never repeat for same key — use random 12-byte nonces
- **Timing attacks**: String comparison, hash verification must use constant-time operations
- **Error messages**: Different errors for "user not found" vs "wrong password" enable enumeration
- **Key storage**: Never hardcode keys — use environment variables, HSMs, or cloud KMS
- **Deprecated algorithms**: Avoid MD5, SHA-1, DES, RC4, ECB mode

## Verification
- All crypto uses established libraries (no custom implementations)
- Nonces are randomly generated and never reused
- Padding oracle tests return negative
- Timing tests show <100ns difference between valid/invalid inputs
- KDF parameters meet OWASP recommendations (PBKDF2 ≥600K iterations, bcrypt cost ≥12)
- No hardcoded keys or secrets in source code
- All sensitive data encrypted at rest and in transit