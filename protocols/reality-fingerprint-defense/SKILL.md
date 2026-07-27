---
name: reality-fingerprint-defense
description: 
category: protocols
tags: [reality-fingerprint-defense]
---

## When to Use
Defend REALITY connections against fingerprinting: uTLS profiles, target selection, traffic hygiene.

## uTLS Fingerprints
- **chrome**: Most common, blends with Chrome traffic
- **firefox**: Good alternative, less suspicious
- **safari**: Apple device traffic pattern
- **random**: Changes per connection — may be suspicious

## Target Selection Criteria
1. High-traffic domain (microsoft.com, apple.com, cloudflare.com)
2. Supports TLS 1.3
3. Has valid certificate
4. Server in same region as proxy server
5. Not blocked by DPI

## Traffic Hygiene
1. Don't send large uploads in burst — pace traffic
2. Use realistic packet sizes (not all 1500-byte MTU)
3. Add random delays between requests
4. Don't maintain too many concurrent connections

## Pitfalls
- **Target must be valid**: If dest is unreachable, REALITY fails
- **Fingerprint mismatch**: Client fingerprint must match real browser
- **Traffic pattern**: Too much traffic to one dest looks suspicious

## Verification
- Wireshark shows ClientHello matching target fingerprint
- No REALITY signature in packet metadata
- Traffic patterns look like normal web browsing