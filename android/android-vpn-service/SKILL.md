---
name: android-vpn-service
description: 
category: android
tags: [android-vpn-service]
---

## When to Use
Use this skill when building VPN clients with Android VpnService: TUN interface, packet routing, per-app tunneling, always-on VPN.

## Core Concepts
- **VpnService**: Android service that creates and manages a VPN interface
- **TUN device**: Virtual network interface for capturing all traffic
- **Packet filter**: Select which apps use the VPN (per-app tunneling)
- **Always-on**: System-managed VPN that auto-reconnects
- **FileDescriptor**: Raw packet read/write on the TUN interface

## Workflow
1. Declare VpnService permission in AndroidManifest
2. Request VPN permission with VpnService.prepare()
3. Build VpnService.Builder with IP/DNS routes
4. Open TUN FileDescriptor
5. Read/write packets in a background thread
6. Forward packets to upstream proxy (WireGuard, Xray, etc.)

## Key Patterns
```kotlin
// AndroidManifest
<service android:name=".VpnTunnelService"
    android:permission="android.permission.BIND_VPN_SERVICE">
    <intent-filter>
        <action android:name="android.net.VpnService" />
    </intent-filter>
</service>

// Build VPN interface
val builder = Builder()
    .setSession("SurfShield")
    .addAddress("10.0.0.2", 32)
    .addRoute("0.0.0.0", 0)
    .addDnsServer("1.1.1.1")
    .setMtu(1500)

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
    builder.setMetered(false)
    builder.addAllowedApplication("com.target.app")
}

val vpnInterface = builder.establish()  // Returns FileDescriptor

// Read packets
val input = FileInputStream(vpnInterface.fileDescriptor)
val buffer = ByteArray(32767)
while (true) {
    val length = input.read(buffer)
    if (length > 0) {
        processPacket(buffer, length)
    }
}
```

## Pitfalls
- **Permission denial**: User must grant VPN permission; handle denial gracefully
- **Battery optimization**: Request exemption for always-on VPN
- **Per-app filtering**: Only available on Android 10+
- **DNS leaks**: Always route DNS through the VPN
- **MTU**: Default 1500; some protocols need lower (1280 for IPv6)

## Verification
- Test with multiple apps to verify per-app routing
- Check for DNS leaks with dnsleaktest.com
- Verify all traffic goes through VPN with Wireshark
- Test always-on with system settings