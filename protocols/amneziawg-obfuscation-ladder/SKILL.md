---
name: amneziawg-obfuscation-ladder
description: Diagnose and configure AmneziaWG (awg) obfuscation parameters as an escalating ladder, correctly separating the vanilla-WireGuard-compatible junk-packet knobs (Jc/Jmin/Jmax) from the server-side-only magic-header knobs (S1/S2/H1-H4) that silently break handshakes against stock WireGuard peers.
category: protocols
tags: [masterclass, networking, wireguard, amneziawg, dpi, censorship-circumvention]
---
# AmneziaWG Obfuscation Ladder

## Overview

AmneziaWG is a fork of WireGuard that adds DPI-evasion parameters to the `[Interface]` section of a `wg-quick` config. Its single most expensive failure mode is that **the parameters split into two classes with completely different compatibility requirements**, and the config file gives you no warning when you cross the line:

| Class | Keys | Requires AmneziaWG on the **server**? | Failure signature if unsupported |
|---|---|---|---|
| Junk packets | `Jc`, `Jmin`, `Jmax` | **No** — works against stock `wg` | none; extra garbage UDP is simply dropped by the peer |
| Magic headers / init padding | `S1`, `S2`, `H1`, `H2`, `H3`, `H4` | **Yes** | interface comes UP, `totalRx()` stays `0` forever |

The second row is the trap. The client reports a healthy tunnel — `Tunnel.State.UP`, route installed, no exception — because bringing up a WireGuard interface is a purely local operation. The handshake is never acknowledged, so nothing is ever received, and the user sees "connected but no internet."

Use this skill whenever an AmneziaWG tunnel comes up but never passes traffic, or when deciding which obfuscation to apply to an unknown server.

## Core Instructions

### 1. Establish server capability before choosing parameters

Never assume. Classify the endpoint first:

- **Known AmneziaWG server** (you or a partner deployed `amneziawg-go` / the `amneziawg` kernel module): the full ladder is available.
- **Commercial WireGuard provider** (Surfshark, Mullvad, ProtonVPN, NordLynx, etc.): stock WireGuard. `S1/S2/H1-H4` are **forbidden**. Only `Jc/Jmin/Jmax` and MTU are yours to tune.
- **Unknown**: treat as stock WireGuard. Probe upward only after a plain handshake succeeds.

Encode this as an explicit boolean in your client model, and make the ladder a function of it:

```kotlin
enum class ObfuscationProfile(val mtu: Int) {
    PLAIN(1420),      // no obfuscation at all
    LIGHT(1400),      // Jc only, small counts
    MEDIUM(1340),     // Jc with wider size spread
    HEAVY(1280),      // Jc maxed; smallest safe MTU
    FULL_AWG(1280);   // + S1/S2/H1-H4  -- AWG servers ONLY

    companion object {
        fun ladder(serverSupportsAwg: Boolean): List<ObfuscationProfile> =
            if (serverSupportsAwg) listOf(PLAIN, LIGHT, MEDIUM, HEAVY, FULL_AWG)
            else listOf(PLAIN, LIGHT, MEDIUM, HEAVY)
    }
}
```

The ladder must be ordered cheapest-first. Obfuscation costs throughput and battery; escalate only when a rung fails.

### 2. Understand what each knob actually does

- `Jc` — **count** of junk packets sent before the real handshake initiation. Range 1-128; useful range **3-10**. Above ~12 you are adding measurable connect latency for no additional evasion.
- `Jmin` / `Jmax` — byte size bounds of each junk packet. Constraint: `Jmin < Jmax` and `Jmax <= 1280`. A wider spread defeats size-histogram classifiers; a fixed size (`Jmin == Jmax - 1`) is itself a fingerprint. Use **50-1000**.
- `S1` / `S2` — bytes of random padding prepended to the handshake **init** and **response** packets. This changes the packet *length*, which is exactly what a WireGuard-length-signature DPI looks for. Constraint: `S1 + 56 != S2` (otherwise init and response become indistinguishable in a way the protocol rejects). Typical `S1=15, S2=25`.
- `H1`-`H4` — the four **magic header** values replacing WireGuard's fixed message-type bytes (`1,2,3,4`) for handshake-init, handshake-response, cookie-reply and transport-data. Must be **distinct**, in range `5..2147483647`, and **identical on both peers**. This is the single strongest signature-killer and the single hardest compatibility requirement.

### 3. Escalate on evidence, not on hope

A rung has failed only when you have waited for a *receive*, not for a state change:

```kotlin
private suspend fun awaitHandshake(tunnel: Tunnel): Boolean {
    val deadline = System.currentTimeMillis() + HANDSHAKE_TIMEOUT_MS  // 12_000L
    while (System.currentTimeMillis() < deadline) {
        val rx = runCatching { backend.getStatistics(tunnel).totalRx() }.getOrDefault(0L)
        if (rx > 0L) return true          // the ONLY proof of a real tunnel
        delay(HANDSHAKE_POLL_MS)          // 300L
    }
    return false
}
```

12 s is the right budget: a WireGuard peer retries initiation at 5 s intervals, so 12 s covers two attempts without stranding the user.

### 4. Cross the ladder with the endpoint matrix

Obfuscation and endpoint reachability are orthogonal failures. Iterate the outer loop over endpoints and the inner loop over profiles, and cap both (`MAX_ENDPOINTS = 3`, `MAX_PROFILES = 3`) so a total blackout fails in ~2 minutes rather than 20:

```
for endpoint in rankedEndpoints.take(3):
    for profile in ladder(serverSupportsAwg).take(3):
        bring up; if awaitHandshake(): persist (endpoint, profile) and return
tear down; report Failure.NoHandshake
```

### 5. Cache the winning combination per network

The combination that works is a property of **the network you are on**, not of the app. A profile that handshakes on home fibre may be blocked on a mobile carrier. Key the cache by network fingerprint plus server:

```kotlin
fun networkFingerprint(cm: ConnectivityManager): String {
    val caps = cm.getNetworkCapabilities(cm.activeNetwork ?: return "offline")
        ?: return "unknown"
    return when {
        caps.hasTransport(TRANSPORT_WIFI)     -> "wifi"
        caps.hasTransport(TRANSPORT_CELLULAR) -> "cell:" + telephony.simOperator
        caps.hasTransport(TRANSPORT_ETHERNET) -> "ethernet"
        else -> "other"
    }
}

val cacheKey = "$fingerprint|${location.id}"   // -> ObfuscationProfile + endpoint
```

On the next connect, try the cached combination **first**, then fall back to the full ladder. This turns a 40 s cold start into a 2 s warm start.

## Proven Recipes

### Stock-WireGuard provider (Surfshark and friends) — junk packets only

```ini
[Interface]
PrivateKey = <client private key>
Address = 10.14.0.2/16
DNS = 162.252.172.57, 149.154.159.92
MTU = 1340
# vanilla-WG compatible: the peer ignores the extra UDP
Jc = 6
Jmin = 64
Jmax = 900

[Peer]
PublicKey = <server public key>
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = 91.90.123.45:51820
PersistentKeepalive = 21
```

### Real AmneziaWG server — full ladder

Both sides must carry identical `S*`/`H*` values. Server `/etc/amnezia/amneziawg/awg0.conf`:

```ini
[Interface]
PrivateKey = <server private key>
ListenPort = 51820
Jc = 5
Jmin = 50
Jmax = 1000
S1 = 15
S2 = 25
H1 = 1148571415
H2 = 1785498317
H3 = 1521827271
H4 = 1972237441
```

Generate the four headers so they are distinct and out of the reserved low range:

```bash
for i in 1 2 3 4; do
  printf 'H%d = %d\n' "$i" "$(shuf -i 5-2147483647 -n 1)"
done
```

### Verifying from the server side

A client-side "UP" proves nothing. Confirm on the server:

```bash
awg show awg0 latest-handshakes   # non-zero epoch == real handshake
awg show awg0 transfer            # rx/tx must both grow
```

If `latest-handshakes` is `0` while the client says connected, you are looking at the `S*`/`H*` mismatch described above.

### PersistentKeepalive on carrier NAT

Iranian and most mobile carriers expire UDP NAT mappings aggressively (30-60 s). Set `PersistentKeepalive = 21`. Higher values cause the tunnel to die silently on screen-off; lower values waste battery for no gain.

## Potential Pitfalls

1. **`S1`/`S2`/`H1`-`H4` against a stock WireGuard server.** The interface comes up, no error is raised, and the tunnel never receives a byte. If your only success criterion is `Tunnel.State.UP`, you will ship this bug. Gate these keys behind an explicit `serverSupportsAwg` flag and make `totalRx() > 0` the success criterion.
2. **Fabricated Android dependency.** AmneziaWG for Android has **no published Maven coordinate**. `implementation("org.amnezia.awg:tunnel:1.x")` does not resolve; any snippet containing it is invented. Vendor the library instead: `git submodule add https://github.com/amnezia-vpn/amneziawg-android third_party/amneziawg-android`, then include its `tunnel` module as a local Gradle project. Runtime classes are `org.amnezia.awg.backend.GoBackend`, `...backend.Tunnel`, `...config.Config.parse(InputStream)`.
3. **MTU left at 1420 with heavy obfuscation.** Junk padding plus the tunnel header pushes packets past the path MTU; TCP handshakes succeed while large responses black-hole, producing "the tunnel works but websites hang." Drop to 1340, then 1280, as you climb the ladder.
4. **Logging the config verbatim.** `wg-quick` configs contain `PrivateKey`. Always redact before any log or crash report: `config.replace(Regex("(PrivateKey\\s*=\\s*)\\S+"), "$1<redacted>")`.
5. **`Jc` set very high as a "stronger" setting.** Junk count is not a security dial. Beyond ~12 packets you add seconds of connect latency and a *new* fingerprint (an unusual burst count) while gaining nothing.
6. **Reusing one cached profile globally.** Caching the winner without the network fingerprint means the app remembers a Wi-Fi-only profile and fails permanently on mobile data.
