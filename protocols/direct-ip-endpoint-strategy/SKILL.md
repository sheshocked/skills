---
name: direct-ip-endpoint-strategy
description: Eliminate DNS as a censorship and failure surface by shipping raw IP:port endpoints, ranking them with a concurrent latency prober across multiple candidate ports, and resolving names only via DoH when a literal IP is genuinely unavailable.
category: protocols
tags: [masterclass, networking, dns, doh, censorship-circumvention, latency]
---
# Direct-IP Endpoint Strategy

## Overview

On a censored network, the DNS lookup is usually the first thing to fail and always the cheapest thing to attack. A hostile resolver can NXDOMAIN your endpoint, poison it to a blackhole, or simply answer slowly enough that your connect timeout fires. Every one of these failures happens *before* your carefully obfuscated tunnel gets a chance to run.

The fix is structural: **treat the hostname as an optional convenience and the IP:port pair as the real identity of an endpoint.** A client that never resolves a name cannot be attacked through the resolver, cannot leak the destination to a plaintext DNS observer, and connects measurably faster because it skips a round trip.

Use this skill when designing an endpoint model for a tunnel client, when a client works on one ISP and fails on another, or when connect times are dominated by resolution rather than handshake.

## Core Instructions

### 1. Model an endpoint as a list, never as a string

A single `Endpoint = host:port` field is the root cause of brittleness. Model it as ranked candidates:

```kotlin
@Serializable
data class SurfLocation(
    val id: String,
    val country: String,
    val city: String,
    val publicKey: String,
    val endpoints: List<String>,   // literal "IP:port", multiple, in preference order
    val dns: List<String>,         // list, not String -- clients need a fallback resolver
    val hostname: String? = null   // for display/diagnostics only; never for connecting
)
```

The `hostname` field exists so a support log can say "nl-ams-3" instead of a number. Nothing in the connect path is allowed to read it.

### 2. Ship several ports per address, not just the canonical one

Many networks throttle or drop UDP/51820 (the WireGuard default) while passing UDP/443 untouched, because 443 must stay open for QUIC. Include the same address on multiple ports and let measurement decide:

```kotlin
val PROBE_PORTS = intArrayOf(443, 80, 1443)
```

443 first because it is the most likely to be unfiltered, 80 as the second-most-permitted, and a high port as the control that tells you whether filtering is port-based at all.

### 3. Rank by measured latency, concurrently

Sequential probing of 9 candidates at 1.5 s each is a 13-second stall. Probe in parallel and sort by median RTT:

```kotlin
suspend fun rank(
    endpoints: List<String>,
    timeoutMs: Int = 1_500,
    attempts: Int = 2
): List<String> = coroutineScope {
    endpoints.map { ep ->
        async(Dispatchers.IO) {
            val (host, port) = ep.substringBeforeLast(':') to ep.substringAfterLast(':').toInt()
            val samples = (1..attempts).mapNotNull { probeOnce(host, port, timeoutMs) }
            ep to (samples.minOrNull() ?: Long.MAX_VALUE)
        }
    }.awaitAll()
     .sortedBy { it.second }
     .filter { it.second != Long.MAX_VALUE }
     .map { it.first }
}

private fun probeOnce(host: String, port: Int, timeoutMs: Int): Long? {
    val started = System.nanoTime()
    return try {
        Socket().use { s ->
            s.connect(InetSocketAddress(InetAddress.getByName(host), port), timeoutMs)
        }
        (System.nanoTime() - started) / 1_000_000
    } catch (_: IOException) { null }
}
```

Two attempts, taking the **minimum**: the minimum is a far better estimate of true path latency than the mean, because network noise only ever adds delay.

`InetAddress.getByName` on a literal IP string does **not** perform a DNS query — it parses. This keeps the prober DNS-free by construction.

### 4. Note what a TCP probe does and does not prove

A successful TCP connect to `ip:443` proves the path to that address and port is open. It does **not** prove UDP on the same port is open, and most tunnels are UDP. Treat the ranking as a *prior*, not a verdict: it orders your attempts, and the real handshake decides. This is still enormously valuable, because it moves the likely-working endpoint to position one.

### 5. When you truly need a name, use DoH

Some providers rotate addresses and publish only hostnames. Resolve over HTTPS so the query is neither visible nor forgeable by the local resolver:

```kotlin
suspend fun resolveDoh(domain: String): List<String> = withContext(Dispatchers.IO) {
    val url = "https://dns.google/resolve?name=$domain&type=A"
    val body = URL(url).openConnection().apply {
        setRequestProperty("Accept", "application/dns-json")
        connectTimeout = 4_000; readTimeout = 4_000
    }.getInputStream().bufferedReader().readText()
    Json.parseToJsonElement(body).jsonObject["Answer"]?.jsonArray
        ?.mapNotNull { it.jsonObject["data"]?.jsonPrimitive?.contentOrNull }
        ?.filter { it.count { c -> c == '.' } == 3 }   // A records only
        ?: emptyList()
}
```

Then **cache the result to disk and prefer the cache on the next launch.** A DoH resolver that is itself blocked must not be a hard dependency; hardcode two providers (`dns.google`, `cloudflare-dns.com`) and accept the first that answers.

### 6. Split-tunnel the resolver, or you will deadlock

If your `AllowedIPs` sends all traffic into the tunnel, a DoH lookup performed *while establishing* the tunnel routes into a tunnel that is not up yet. Perform all resolution **before** `Backend.setState(UP)`, or explicitly exclude the DoH resolver addresses from `AllowedIPs`.

## Proven Recipes

### Endpoint bundle in the shipped asset

```json
{
  "id": "nl-ams-1",
  "country": "Netherlands",
  "city": "Amsterdam",
  "publicKey": "AbCd...=",
  "endpoints": [
    "185.93.180.11:443",
    "185.93.180.11:80",
    "185.93.180.11:1443",
    "185.93.180.12:443"
  ],
  "dns": ["1.1.1.1", "8.8.8.8"]
}
```

A second address for the same city is what turns a single-server outage from an app-wide failure into an invisible retry.

### Connect sequence

```kotlin
val ranked = EndpointProber.rank(location.endpoints)          // ~1.5 s, concurrent
val ordered = listOfNotNull(cache.lastGood(key)) + ranked     // warm start first
for (endpoint in ordered.distinct().take(MAX_ENDPOINTS)) {
    for (profile in obfuscationLadder.take(MAX_PROFILES)) {
        if (tryHandshake(endpoint, profile)) {
            cache.remember(key, endpoint, profile); return Success
        }
    }
}
return Failure.NoHandshake
```

### Excluding local and private ranges from the tunnel

Routing `0.0.0.0/0` breaks LAN printers, Chromecast and captive portals. Use an explicit "all except private" CIDR set in `AllowedIPs`:

```
AllowedIPs = 0.0.0.0/5, 8.0.0.0/7, 11.0.0.0/8, 12.0.0.0/6, 16.0.0.0/4,
  32.0.0.0/3, 64.0.0.0/2, 128.0.0.0/3, 160.0.0.0/5, 168.0.0.0/6,
  172.0.0.0/12, 172.32.0.0/11, 172.64.0.0/10, 172.128.0.0/9, 173.0.0.0/8,
  174.0.0.0/7, 176.0.0.0/4, 192.0.0.0/9, 192.128.0.0/11, 192.160.0.0/13,
  192.169.0.0/16, 192.170.0.0/15, 192.172.0.0/14, 192.176.0.0/12,
  192.192.0.0/10, 193.0.0.0/8, 194.0.0.0/7, 196.0.0.0/6, 200.0.0.0/5,
  208.0.0.0/4
```

This is `0.0.0.0/0` minus `10/8`, `172.16/12`, `192.168/16` and `127/8`, expressed as the minimal prefix set.

### Keeping domestic traffic off the tunnel

For latency and for cost, route in-country ranges directly by placing them in the VPN service's **disallowed** set (or, on platforms without one, by omitting them from `AllowedIPs`). Maintain that CIDR list as data in your asset bundle, never as a constant in code — it changes monthly.

## Potential Pitfalls

1. **A single `Endpoint` string in the config.** One blocked address, and the app is dead with no diagnostic. The list is the feature.
2. **Probing sequentially.** Nine candidates at 1.5 s serialised is a 13-second UI stall that users read as "broken." `async` + `awaitAll` makes it 1.5 s total.
3. **Treating a TCP probe as proof of a working UDP tunnel.** Ranking orders attempts; only `totalRx() > 0` confirms a tunnel.
4. **`dns` typed as `String`.** The moment the first resolver is blocked you need a second, and a schema change ripples through the whole client. Make it `List<String>` from day one.
5. **Resolving DoH after the tunnel is up with `AllowedIPs = 0.0.0.0/0`.** The lookup routes into the half-built tunnel and hangs until timeout. Resolve first, or exclude the resolver IPs.
6. **Averaging probe samples.** Network noise is one-sided; the mean is biased upward by jitter. Take the minimum.
7. **No disk cache of DoH answers.** Your app now fails whenever the DoH provider itself is filtered — exactly the situation the app exists for.
