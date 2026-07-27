---
name: tuic-protocol
description: 
category: protocols
tags: [tuic-protocol]
---

## When to Use
Deploy TUIC v5 QUIC-based proxy for zero-RTT connections and UDP relay.

## Server Config
```json
{
    "relay": {
        "token": "YOUR_TOKEN",
        "users": {
            "USER_UUID": "PASSWORD"
        },
        "congestion_control": "bbr"
    },
    "server": [
        "[::]:443"
    ],
    "certificate": "/path/fullchain.pem",
    "private_key": "/path/privkey.pem"
}
```

## Pitfalls
- **TUIC v5**: Use v5 config format, not v4
- **UDP**: QUIC requires UDP connectivity
- **Congestion**: BBR recommended for best performance

## Verification
- Test with tuic-client
- Check zero-RTT connection
- Verify UDP relay works