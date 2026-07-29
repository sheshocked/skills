---
name: android-vpn-service
description: Establish a production-grade Kotlin VpnService, managing local TUN interfaces and JNI loop mappings.
category: android
tags: [vpnservice, android-vpn, tun-interface, packet-routing, kotlin, surfshield]
---

# Android Vpn Service

## When to Use
Use when building custom Android VPN clients (such as SurfShield) to capture all network traffic, route IP packets, and pass them into low-level protocol engines (WireGuard, Xray).

## Prerequisites
- Android SDK 29+ (Q) for per-app dynamic filtering.
- Native C/C++ or Rust core compiled for targeted device architectures.

## Workflow
1. Declare the VPN service and BIND_VPN_SERVICE permission in the manifest.
2. Request VPN authorization from the user using `VpnService.prepare()`.
3. Build the virtual TUN interface configuration using `VpnService.Builder` (specifying address, route, DNS, and MTU limits).
4. Establish the file descriptor, execute JNI bridging, and handle read/write streams inside background worker threads.
5. Setup clean service destruction and interface closure hooks.

## Key Patterns

### Kotlin VPN Service Wrapper (VpnCoreService.kt)
```kotlin
package com.surfshield.vpn

import android.content.Intent
import android.net.VpnService
import android.os.ParcelFileDescriptor
import android.util.Log
import java.io.IOException

class VpnCoreService : VpnService(), Runnable {
    private var vpnThread: Thread? = null
    private var vpnInterface: ParcelFileDescriptor? = null

    companion object {
        init {
            System.loadLibrary("surfshield_native")
        }
    }

    private external fun startNativeCore(tunFd: Int)
    private external fun stopNativeCore()

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        vpnThread = Thread(this, "VpnWorkerThread").apply { start() }
        return START_STICKY
    }

    override fun run() {
        try {
            establishVpnInterface()
            val fd = vpnInterface?.fd ?: return
            
            // Pass the native File Descriptor directly to Rust/C++
            startNativeCore(fd)
        } catch (e: Exception) {
            Log.e("VpnCore", "Error during VPN execution: ${e.message}")
        }
    }

    private fun establishVpnInterface() {
        vpnInterface = Builder()
            .setSession("SurfShieldCore")
            .setMtu(1360) // Critical: Prevents packet fragmentation on Iranian mobile networks (MCI/Irancell)
            .addAddress("10.0.0.2", 32)
            .addRoute("0.0.0.0", 0)
            .addDnsServer("1.1.1.1")
            .setBlocking(true)
            .establish()
    }

    override fun onDestroy() {
        stopNativeCore()
        try {
            vpnInterface?.close()
        } catch (e: IOException) {
            Log.e("VpnCore", "Failed to close TUN: ${e.message}")
        }
        vpnThread?.interrupt()
        super.onDestroy()
    }
}
```

## Pitfalls
- **Cellular MTU drops:** Enforcing a default MTU of 1500 triggers packet drops on restricted cellular networks. Restrict MTU to `1360` or `1400` to secure connection stability.
- **DNS Leakage:** If you do not specify DNS servers directly inside the builder, Android defaults to DNS servers provided by the mobile operator, leaking all requests. Always set static DNS (e.g. `1.1.1.1`).

## Verification
- Verify TUN routing: run `adb shell ip addr show tun0` and verify the gateway is mapped to `10.0.0.2`.
- Inspect logs to confirm zero `Bad file descriptor` alerts during reconnection cycles.

