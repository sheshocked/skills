---
name: wireguard-android
description: Implement WireGuard client configurations and state management in Android apps using the official Go-backend wrapper.
category: android
tags: [wireguard, android-vpn, go-backend, kotlin, integration]
---

# WireGuard Android Integration Masterclass

## When to Use
Use when embedding low-latency, kernel-style WireGuard or AmneziaWG connections inside Android applications.

## Prerequisites
- Library dependency `com.wireguard.android:wireguard-android-sdk:1.0.2023` added to Gradle.

## Workflow
1. Implement the `Tunnel` interface to observe state transitions.
2. Initialize `GoBackend` or `Wintun` engine components.
3. Build the profile configuration (address, key pairs, endpoints, keepalives).
4. Command the backend to transition state to `UP`.

## Key Patterns

### Kotlin Integration Interface (WgTunnelManager.kt)
```kotlin
package com.surfshield.vpn

import android.content.Context
import com.wireguard.android.backend.Backend
import com.wireguard.android.backend.GoBackend
import com.wireguard.android.backend.Tunnel
import com.wireguard.config.Config
import com.wireguard.config.Interface
import com.wireguard.config.Peer

class WgTunnelManager(private val context: Context) {
    private val backend: Backend = GoBackend(context)
    private val tunnel = AppTunnel()

    class AppTunnel : Tunnel {
        override fun getName(): String = "surfshield_tunnel"
        override fun onStateChange(state: Tunnel.State) {
            Log.d("WgTunnel", "State transitioned to: $state")
        }
    }

    fun connect(privateKey: String, peerPublicKey: String, endpoint: String) {
        val wgInterface = Interface.Builder()
            .addAddress("10.0.0.2/32")
            .setPrivateKey(privateKey)
            .addDnsServer("1.1.1.1")
            .build()

        val peer = Peer.Builder()
            .addAllowedIP("0.0.0.0/0")
            .setEndpoint(endpoint)
            .setPublicKey(peerPublicKey)
            .setPersistentKeepalive(25)
            .build()

        val config = Config.Builder()
            .setInterface(wgInterface)
            .addPeer(peer)
            .build()

        Thread {
            backend.setState(tunnel, Tunnel.State.UP, config)
        }.start()
    }

    fun disconnect() {
        Thread {
            backend.setState(tunnel, Tunnel.State.DOWN, null)
        }.start()
    }
}
```

## Pitfalls
- **Main thread execution blocks:** State transitions trigger socket operations. Never call `backend.setState` from the main thread; always wrap it in coroutines or background executor threads.
- **Port Conflict in User-Space:** GoBackend binds sockets to random ports locally. Ensure proper exception handlers intercept bind conflicts.

## Verification
- Call `backend.getStatistics(tunnel).handshakes()` after 5 seconds to confirm handshake counts are incrementing.
- Perform speed tests to verify interface latency.
