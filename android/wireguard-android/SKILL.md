---
name: wireguard-android
description: Integrate the official WireGuard Go-backend library and manage VPN tunnels in Android applications.
category: android
tags: [wireguard, android-vpn, go-backend, kotlin, integration]
---

# Wireguard Android

## When to Use
Use when embedding low-latency, kernel-style WireGuard or AmneziaWG connections inside Android applications.

## Prerequisites
- Library dependency `com.wireguard.android:wireguard-android-sdk` added to build configurations.

## Workflow
1. Implement the `Tunnel` interface to observe state transitions.
2. Initialize `GoBackend` using project contexts.
3. Construct the profile configuration specifying private keys, dynamic DNS addresses, and keepalive metrics.
4. Execute state changes in background threads.

## Key Patterns

### Kotlin Connection Manager (WgTunnelManager.kt)
```kotlin
package com.surfshield.vpn

import android.content.Context
import android.util.Log
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

        // State changes trigger network calls; execute in background threads
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
- **UI thread blockages:** State changes block execution during socket binds. Always execute `setState` outside the UI thread.
- **Port Collisions:** GoBackend binds sockets dynamically. Ensure correct exception interception if ports conflict.

## Verification
- Audit handshakes: verify `backend.getStatistics(tunnel).handshakes()` counts increment after 5 seconds.

