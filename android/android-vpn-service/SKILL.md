---
name: android-vpn-service
description: Implement production-grade Android VPNService classes in Kotlin, handling local TUN interfaces and JNI loops.
category: android
tags: [android-vpn, vpnservice, kotlin, tun-interface, ndk, multithreading]
---

# Android VpnService implementation Masterclass

## When to Use
Use when building Android VPN clients (e.g. SurfShield) to intercept system IP traffic and route packets programmatically into native protocol engines (WireGuard, Xray).

## Prerequisites
- Android SDK 29+ (Q) for dynamic API configurations.
- NDK toolchain compiled for aarch64.

## Workflow
1. Declare dynamic VpnService binds inside the Android Manifest.
2. Initialize runtime check configurations.
3. Configure the virtual TUN interface parameters using `VpnService.Builder`.
4. Run the socket read/write loops inside background worker threads.

## Key Patterns

### VPN Service Implementation (VpnCoreService.kt)
```kotlin
package com.surfshield.vpn

import android.content.Intent
import android.net.VpnService
import android.os.ParcelFileDescriptor
import android.util.Log
import java.io.FileInputStream
import java.io.FileOutputStream
import java.io.IOException

class VpnCoreService : VpnService(), Runnable {
    private var vpnThread: Thread? = null
    private var vpnInterface: ParcelFileDescriptor? = null

    companion object {
        init {
            System.loadLibrary("surfshield_native")
        }
    }

    // Native engine declaration
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
            
            // Pass FD directly to Rust/C++ network engine
            startNativeCore(fd)
        } catch (e: Exception) {
            Log.e("VpnCoreService", "Error during VPN execution: ${e.message}")
        }
    }

    private fun establishVpnInterface() {
        vpnInterface = Builder()
            .setSession("SurfShieldCore")
            .setMtu(1360) // Tune to avoid fragmentation on MCI/Irancell
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
            Log.e("VpnCoreService", "Failed to close TUN: ${e.message}")
        }
        vpnThread?.interrupt()
        super.onDestroy()
    }
}
```

## Pitfalls
- **MTU over-allocation:** Cellular interfaces drop packet fragments above 1400 bytes on restricted networks. Enforce `setMtu(1360)` to secure throughput.
- **DNS Leakage:** If DNS addresses are not explicitly added using `addDnsServer()`, Android will fallback to mobile ISP DNS, exposing target hosts.

## Verification
- Inspect interface: `adb shell ip addr show tun0` should display addresses matching `10.0.0.2`.
- Audit logs: verify no `IOException: Bad file descriptor` trace alerts occur during service teardowns.
