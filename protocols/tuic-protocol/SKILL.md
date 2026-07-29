---
name: tuic-protocol
description: Deploy TUIC proxy protocol over HTTP/3 QUIC transport, optimizing socket latency.
category: protocols
tags: [tuic, quic, http3, udp-performance, proxy]
---

# Tuic Protocol

## When to Use
Use to run lightweight, highly multiplexed UDP/TCP streams inside HTTP/3 connections.

## Prerequisites
- TUIC binary compiled and installed.

## Workflow
1. Configure server certificate and listening UDP port.
2. Select Congestion Control settings (BBR/cubic).
3. Connect with client wrapper mapping local SOCKS to remote server.

## Key Patterns
```json
// TUIC Server Config
{
  "server": "[::]:8443",
  "users": {
    "UUID-KEY-HERE": "my_password"
  },
  "certificate": "/path/to/cert.pem",
  "private_key": "/path/to/privkey.pem",
  "congestion_control": "bbr",
  "alpn": ["h3"]
}
```

## Pitfalls
- **Firewall blocking:** Ensure UDP ports match open rules inside security policies.
- **Certificate validity:** TUIC checks timestamps strictly; ensure system times are synchronized.

## Verification
- Verify start validation logs.
- Perform connectivity check from client dashboard.
