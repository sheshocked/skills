---
name: wireguard-android
description: Integrate the official WireGuard Android Go/Rust library for modern VPN tunnel implementation.
category: android
tags: [wireguard, android-wireguard, vpn-tunnel, amneziawg, kotlin]
---

# Wireguard Android

## When to Use
Use when embedding WireGuard or AmneziaWG engines inside Android apps to achieve low-latency obfuscated connections.

## Prerequisites
- WireGuard Android library dependency in `build.gradle.kts`.

## Workflow
1. Initialize the backend engine (Go backend or User-space TUN).
2. Format the peer and interface configuration keys.
3. Apply the WireGuard state change.

## Key Patterns
```kotlin
import com.wireguard.android.backend.Backend
import com.wireguard.android.backend.GoBackend
import com.wireguard.android.backend.Tunnel

class WgTunnel : Tunnel {
    override fun getName(): String = "wireguard_tunnel"
    override fun onStateChange(state: Tunnel.State) {
        // Handle connection states
    }
}

val backend = GoBackend(context)
val tunnel = WgTunnel()
val config = Tunnel.Config.Builder()
    .setInterfaceAddress("10.0.0.2/32")
    .setPrivateKey("PEER_PRIVATE_KEY")
    .addPeer("SERVER_PUBLIC_KEY", "185.71.219.72:51820", "0.0.0.0/0")
    .build()
backend.setState(tunnel, Tunnel.State.UP, config)
```

## Pitfalls
- **SELinux policy denials:** Ensure native binary permissions match target requirements.
- **Port collisions:** Ensure dynamic ports do not conflict with local webservers.

## Verification
- Test handshakes via `backend.getStatistics(tunnel).handshakes`.
- Verify MTU throughput with large packet pings.
