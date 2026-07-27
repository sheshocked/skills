---
name: wireguard-android
description: 
category: android
tags: [wireguard-android]
---

## When to Use
Use this skill when integrating WireGuard/AmneziaWG tunnels in Android apps: Go backend compilation, config management, obfuscation.

## Architecture
```
Kotlin UI Layer
  ↓ JNI calls
Go WireGuard Library (wireguard-go)
  ↓
TUN Interface (Android VpnService)
```

## Workflow
1. Add wireguard-go as submodule or dependency
2. Compile Go library for Android ABIs (arm64, armeabi-v7a, x86_64)
3. Create JNI bridge between Kotlin and Go
4. Parse WireGuard config files
5. Start tunnel through VpnService

## Key Patterns
```kotlin
// JNI Bridge
external fun wgTurnOn(goVpnService: Int, tunFd: Int, settings: String): Int
external fun wgTurnOff(id: Int)
external fun wgGetSocketV4(): Int
external fun wgSetConfig(id: Int, settings: String): Int

// Config parser
fun parseWgConfig(config: String): TunnelConfig {
    val lines = config.lines()
    var privateKey = ""
    var address = ""
    var dns = ""
    val peers = mutableListOf<Peer>()

    var currentPeer: Peer? = null
    for (line in lines) {
        when {
            line.startsWith("PrivateKey = ") -> privateKey = line.substringAfter("= ")
            line.startsWith("Address = ") -> address = line.substringAfter("= ")
            line.startsWith("DNS = ") -> dns = line.substringAfter("= ")
            line.startsWith("[Peer]") -> currentPeer = Peer().also { peers.add(it) }
            line.startsWith("PublicKey = ") -> currentPeer?.publicKey = line.substringAfter("= ")
            line.startsWith("Endpoint = ") -> currentPeer?.endpoint = line.substringAfter("= ")
            line.startsWith("AllowedIPs = ") -> currentPeer?.allowedIPs = line.substringAfter("= ")
        }
    }
    return TunnelConfig(privateKey, address, dns, peers)
}
```

## AmneziaWG Differences
- Added fields: Jc, Jmin, Jmax (junk packet count/min/max), S1, S2 (handshake init/padding), H1-H4 (magic headers)
- These fields are added to WireGuard config to defeat DPI fingerprinting
- Go library needs AmneziaWG patches applied

## Pitfalls
- **ABI coverage**: Must build for arm64-v8a, armeabi-v7a, and x86_64
- **Key rotation**: Don't rotate keys on connect — keep them stable
- **Config encryption**: Don't store plaintext configs — use EncryptedSharedPreferences
- **MTU**: WireGuard default MTU is 1420; adjust for transport overhead

## Verification
- Test with official WireGuard client to verify config compatibility
- Check packet capture shows encrypted WireGuard packets
- Verify DNS resolution through tunnel
- Test handoff between WiFi and cellular