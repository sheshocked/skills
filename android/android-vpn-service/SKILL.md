---
name: android-vpn-service
description: Establish a local TUN interface and route packet streams programmatically on Android using JNI/VPNService.
category: android
tags: [vpnservice, android-vpn, tun-interface, packet-routing, kotlin]
---

# Android Vpn Service

## When to Use
Use this skill when developing custom Android VPN clients (e.g., SurfShield, Aethery) that require kernel-level IP packet capture using Android's native `VpnService` API.

## Prerequisites
- Android SDK 29+ (Q+) for per-app filtering
- Native C/C++ or Rust core for high-throughput packet processing

## Workflow
1. Declare the VPN service and permission in `AndroidManifest.xml`.
2. Request user consent using `VpnService.prepare()`.
3. Configure the local TUN interface using `VpnService.Builder` (MTU, IP, Routing, DNS).
4. Establish the file descriptor and handle packet read/write loop in a background thread.
5. Gracefully handle service destruction and interface teardown.

## Key Patterns
```kotlin
// AndroidManifest.xml
<service
    android:name=".MyVpnService"
    android:permission="android.permission.BIND_VPN_SERVICE"
    android:exported="false">
    <intent-filter>
        <action android:name="android.net.VpnService" />
    </intent-filter>
</service>

// Core VPN Service implementation
class MyVpnService : VpnService() {
    private var vpnInterface: ParcelFileDescriptor? = null

    fun startVpn() {
        val builder = Builder()
            .setSession("SurfShieldCore")
            .setMtu(1400)
            .addAddress("10.0.0.2", 32)
            .addRoute("0.0.0.0", 0)
            .addDnsServer("1.1.1.1")
            .setBlocking(true)
        
        vpnInterface = builder.establish()
        val fd = vpnInterface?.fd ?: return
        
        // Start native worker thread passing the fd
        startNativeEngine(fd)
    }

    override fun onDestroy() {
        vpnInterface?.close()
        super.onDestroy()
    }
}
```

## Pitfalls
- **MTU size issues:** MTU above 1420 causes packet fragmentation over cellular networks (MCI/Irancell). Use `setMtu(1360)` or `1400`.
- **Memory leaks in background loops:** Ensure the file descriptor loop closes properly when the service is stopped.

## Verification
- Run `adb shell dumpsys connectivity vpn` to verify the active VPN session.
- Check packet routing using `ping -I tun0 1.1.1.1` in adb shell.
