---
name: udp-over-tcp-tuning
description: 
category: protocols
tags: [udp-over-tcp-tuning]
---

## When to Use
Optimize UDP-over-TCP tunneling: BBR congestion control, QUIC tuning, latency optimization.

## BBR Enablement
```bash
# Check current congestion control
sysctl net.ipv4.tcp_congestion_control

# Enable BBR
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```

## QUIC Tuning
```bash
# UDP buffer sizes
sysctl -w net.core.rmem_max=7500000
sysctl -w net.core.wmem_max=7500000
```

## Pitfalls
- **BBR requires kernel 4.9+**
- **UDP firewalls**: Many block UDP — may need UDP-over-TCP
- **Buffer sizes**: Too small causes packet loss under load

## Verification
- Test throughput with iperf3
- Check BBR is active with `ss -tin`
- Measure latency improvement