---
name: game-networking
description: - Client-server architecture for online games
category: threed
tags: [game-networking]
---

## When to Use
- Multiplayer game development (real-time or turn-based)
- Client-server architecture for online games
- Network synchronization and interpolation
- Lag compensation and prediction systems
- Scalable server infrastructure for game backends

## Core Concepts
- Client-Server model: authoritative server prevents cheating
- Tick rate: server update frequency (20-60 ticks/second typical)
- Interpolation: smooth visual movement between network snapshots
- Prediction: client predicts outcomes, server corrects on mismatch
- Replication: syncing game state from server to clients
- RPC (Remote Procedure Call): client→server (unreliable) or server→client
- Lag compensation: server rewinds state to client's view time
- Netcode types: deterministic lockstep, state sync, snapshot interpolation

## Workflow
```
# Architecture decision
1. Define authoritative model (server-authoritative recommended)
2. Choose network model:
   - Small-scale (4-8 players): UDP with custom protocol
   - Large-scale: GameLift/Agones + dedicated servers
   - Turn-based: HTTP/WebSocket + cloud functions
3. Implement basic movement replication first
4. Add interpolation and prediction for smooth feel
5. Add lag compensation for hit detection
6. Stress test with simulated latency (200ms+, packet loss 5-10%)
7. Deploy with matchmaking and session management
```

## Key Patterns
```csharp
// Unity Netcode for GameObjects (NGO)
using Unity.Netcode;

public class PlayerMovement : NetworkBehaviour
{
    private NetworkVariable<float> health = new NetworkVariable<float>(
        100f, NetworkVariableReadPermission.Everyone,
        NetworkVariableWritePermission.Server
    );

    public override void OnNetworkSpawn()
    {
        if (!IsOwner) return; // Only control own character
        // Subscribe to health changes
        health.OnValueChanged += OnHealthChanged;
    }

    [ServerRpc]
    public void TakeDamageServerRpc(float amount)
    {
        health.Value -= amount;
        if (health.Value <= 0)
            GetComponent<NetworkObject>().Despawn();
    }

    void OnHealthChanged(float oldVal, float newVal)
    {
        healthBar.fillAmount = newVal / 100f;
    }
}
```

```python
# Simple authoritative server (Python asyncio)
import asyncio
import struct

class GameServer:
    def __init__(self, host='0.0.0.0', port=7777):
        self.players = {}
        self.tick_rate = 20
        self.tick_interval = 1.0 / self.tick_rate

    async def start(self):
        transport, protocol = await asyncio.get_event_loop().create_datagram_endpoint(
            lambda: self, local_addr=(self.host, self.port)
        )
        asyncio.create_task(self.game_loop())

    async def game_loop(self):
        while True:
            start = asyncio.get_event_loop().time()
            self.process_inputs()
            self.simulate()
            self.broadcast_state()
            elapsed = asyncio.get_event_loop().time() - start
            await asyncio.sleep(max(0, self.tick_interval - elapsed))

    def process_inputs(self):
        for pid, player in self.players.items():
            if player.pending_inputs:
                inp = player.pending_inputs.pop(0)
                # Validate input on server
                if self.validate_input(player, inp):
                    player.position += inp['direction'] * self.tick_interval * player.speed

    def broadcast_state(self):
        state = self.serialize_state()
        for player in self.players.values():
            self.send_to(player.addr, state)
```

Interpolation client:
```python
# Snapshot interpolation with buffering
class InterpolationBuffer:
    def __init__(self, delay_ms=100):
        self.buffer = []
        self.delay = delay_ms / 1000.0

    def add_snapshot(self, timestamp, state):
        self.buffer.append((timestamp, state))
        # Keep last 2 seconds
        self.buffer = [s for s in self.buffer if timestamp - s[0] < 2.0]

    def interpolate(self, render_time):
        # Find two snapshots surrounding render_time
        for i in range(len(self.buffer) - 1):
            t0, s0 = self.buffer[i]
            t1, s1 = self.buffer[i + 1]
            if t0 <= render_time <= t1:
                alpha = (render_time - t0) / (t1 - t0)
                return {k: s0[k] * (1 - alpha) + s1[k] * alpha for k in s0}
        return self.buffer[-1][1] if self.buffer else {}
```

## Pitfalls
- Authoritative server is essential — client-authoritative is trivially exploitable
- Raw UDP needs reliable ordered channels on top (sequence numbers, acks)
- Tick rate mismatch between client and server causes rubber-banding
- Interpolation delay adds visual lag — balance smoothness vs responsiveness
- State synchronization bandwidth: delta compression essential for >4 players
- Connection migration after server restart is complex — design for it early

## Verification
- Simulate 200ms latency + 5% packet loss with tc/netem
- Verify server-side validation catches speed hacks and teleportation
- Profile network bandwidth per client (< 10KB/s for most games)
- Test reconnection: client disconnects and reconnects within 30 seconds
- Load test: run 100 concurrent players on target hardware