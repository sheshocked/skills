---
name: incident-response
description: 
category: security
tags: [incident-response]
---

## When to Use
Use this skill when responding to a security incident — containment, evidence preservation, timeline building, eradication, and post-incident review.

## Core Concepts
- **Containment**: Stop the bleeding — isolate affected systems without destroying evidence
- **Evidence preservation**: Capture forensic artifacts before they're overwritten
- **Timeline**: Reconstruct attacker actions chronologically from logs and artifacts
- **Eradication**: Remove attacker persistence and close the initial access vector
- **Recovery**: Restore systems to known-good state
- **Post-incident**: Root cause analysis and prevention measures

## Workflow
1. **Triage**: Assess scope, severity, and type of incident
2. **Containment**: Isolate affected hosts (network quarantine, not shutdown)
3. **Evidence collection**: Memory dumps, disk images, log preservation
4. **Timeline reconstruction**: Correlate logs across systems
5. **Eradication**: Remove malware, close vulnerabilities, rotate credentials
6. **Recovery**: Restore from clean backups, verify system integrity
7. **Post-mortem**: Document root cause, timeline, and prevention measures

## Key Patterns

### Initial Triage Checklist
```bash
#!/bin/bash
# incident_triage.sh — collect initial forensic data

INCIDENT_DIR="/evidence/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$INCIDENT_DIR"

# System info
uname -a > "$INCIDENT_DIR/uname.txt"
hostname > "$INCIDENT_DIR/hostname.txt"
date > "$INCIDENT_DIR/date.txt"
uptime > "$INCIDENT_DIR/uptime.txt"

# Network connections
ss -tulnp > "$INCIDENT_DIR/listening_ports.txt"
ss -anp > "$INCIDENT_DIR/established_connections.txt"
cat /proc/net/tcp > "$INCIDENT_DIR/proc_net_tcp.txt"
cat /proc/net/udp > "$INCIDENT_DIR/proc_net_udp.txt"

# Running processes
ps auxwwf > "$INCIDENT_DIR/ps_full.txt"
ps -eo pid,ppid,user,args --sort=-rss > "$INCIDENT_DIR/ps_by_memory.txt"

# Network interfaces
ip addr > "$INCIDENT_DIR/ip_addr.txt"
ip route > "$INCIDENT_DIR/ip_route.txt"
arp -a > "$INCIDENT_DIR/arp.txt"

# Users
who > "$INCIDENT_DIR/who.txt"
w > "$INCIDENT_DIR/w.txt"
last -50 > "$INCIDENT_DIR/last_logins.txt"
cat /etc/passwd > "$INCIDENT_DIR/passwd.txt"
cat /etc/shadow > "$INCIDENT_DIR/shadow.txt" 2>/dev/null

# Crontabs
for user in $(cut -d: -f1 /etc/passwd); do
  crontab -l -u "$user" 2>/dev/null > "$INCIDENT_DIR/crontab_${user}.txt"
done
ls -la /etc/cron.* > "$INCIDENT_DIR/cron_dirs.txt"
cat /etc/crontab > "$INCIDENT_DIR/etc_crontab.txt"

# File system
df -h > "$INCIDENT_DIR/disk_usage.txt"
mount > "$INCIDENT_DIR/mounts.txt"
lsof > "$INCIDENT_DIR/open_files.txt" 2>/dev/null

# Recent modifications
find / -mtime -1 -type f 2>/dev/null > "$INCIDENT_DIR/recent_files.txt"
find /tmp -type f -executable 2>/dev/null > "$INCIDENT_DIR/tmp_executables.txt"

# Memory dump (if possible)
if command -v avml &> /dev/null; then
  avml "$INCIDENT_DIR/memory.lime"
elif [ -f /proc/kcore ]; then
  dd if=/proc/kcore of="$INCIDENT_DIR/kcore.dump" bs=1M count=100 2>/dev/null
fi

echo "Evidence collected in $INCIDENT_DIR"
```

### Memory Acquisition (Linux)
```bash
# Using LiME (Linux Memory Extractor)
# Compile for running kernel
insmod lime-$(uname -r).ko path="$INCIDENT_DIR/memory.lime" format=lime

# Alternative: AVML (Microsoft's user-space acquisition)
avml /tmp/memory.lime

# Alternative: /proc/meminfo + dd (quick but incomplete)
dd if=/dev/mem of="$INCIDENT_DIR/mem.dump" bs=1M count=4096 2>/dev/null

# Analyze with Volatility
vol3 -f memory.lime linux.pslist
vol3 -f memory.lime linux.bash  # Recover bash history from memory
vol3 -f memory.lime linux.check_syscall  # Detect syscall hooking
vol3 -f memory.lime linux.malfind  # Detect injected code
```

### Log Correlation Timeline
```bash
#!/bin/bash
# Build incident timeline from multiple log sources

echo "=== Building Timeline ==="

# Extract auth events
journalctl --since "2024-01-15 08:00" --until "2024-01-15 12:00" \
  -u sshd --no-pager | \
  awk '{print $1, $2, $3, $5, $6}' > /tmp/auth_events.txt

# Extract web server access
awk '$4 >= "[15/Jan/2024:08:00" && $4 <= "[15/Jan/2024:12:00"' \
  /var/log/apache2/access.log > /tmp/web_events.txt

# Extract application logs
grep -E "2024-01-15 (0[8-9]|1[0-2]):" /var/log/app/app.log > /tmp/app_events.txt

# Merge and sort by timestamp
cat /tmp/auth_events.txt /tmp/web_events.txt /tmp/app_events.txt | \
  sort -k1,2 > /tmp/unified_timeline.txt

# Extract IOCs from timeline
echo "=== Suspicious IPs ==="
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' /tmp/unified_timeline.txt | \
  sort | uniq -c | sort -rn | head -20

echo "=== Suspicious Commands ==="
grep -E '(curl|wget|nc|ncat|bash -i|/dev/tcp|python.*socket)' /tmp/unified_timeline.txt
```

### Containment Actions
```bash
#!/bin/bash
# containment.sh — isolate compromised host

HOST=$1

# Network quarantine — block all traffic except management
iptables -F
iptables -A INPUT -s 10.10.10.0/24 -j ACCEPT  # Management VLAN only
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A OUTPUT -d 10.10.10.0/24 -j ACCEPT  # Allow responses to management
iptables -P INPUT DROP
iptables -P OUTPUT DROP

# Alternative: move to quarantine VLAN (switch port change)
# On switch: switchport access vlan 999

# Kill suspicious processes (if identified)
# First capture the PID, then kill
SUSPICIOUS_PIDS=$(ss -tulnp | grep -v "sshd\|systemd\|cron" | awk '{print $NF}' | grep -oP '\d+' | sort -u)
for pid in $SUSPICIOUS_PIDS; do
  echo "[QUARANTINE] Killing suspicious PID: $pid"
  kill -9 "$pid" 2>/dev/null
done

# Disable user accounts that may be compromised
passwd -l compromised_user

# Block attacker IPs at network level
for ip in $(cat /tmp/attacker_ips.txt); do
  iptables -A INPUT -s "$ip" -j DROP
done

echo "Host quarantined. Evidence preserved. Begin forensic analysis."
```

### Post-Incident Report Template
```markdown
# Incident Report: [INCIDENT ID]

## Executive Summary
- **Date**: YYYY-MM-DD
- **Severity**: Critical/High/Medium/Low
- **Impact**: [Systems affected, data exposed, business impact]
- **Duration**: [Time of detection to containment]
- **Root Cause**: [Initial access vector]

## Timeline
| Time (UTC) | Event | Source |
|---|---|---|
| 08:00 | Initial access via phishing email | mail logs |
| 08:15 | Malware execution on workstation | EDR |
| 08:30 | Lateral movement to file server | network logs |
| 09:00 | Data exfiltration detected | IDS alerts |
| 09:15 | Host quarantined | IR team |

## Indicators of Compromise (IOCs)
- **IPs**: 203.0.113.45, 198.51.100.22
- **Domains**: evil-c2.example.com
- **File hashes**: abc123... (malware), def456... (dropper)
- **Mutexes**: MalwareMutex123

## Root Cause Analysis
[Detailed explanation of how the incident occurred]

## Remediation Actions
1. Patched CVE-2024-XXXXX on all systems
2. Rotated all credentials
3. Implemented additional email filtering rules
4. Added network segmentation for file servers

## Lessons Learned
- Detection time was too long — implement better alerting
- Backup verification was inadequate — test restores monthly
- Incident response playbooks need updating
```

## Pitfalls
- **Do NOT shut down**: Shutting down destroys volatile evidence (memory, network connections)
- **Preserve order**: Collect memory before disk; volatile before non-volatile
- **Chain of custody**: Document who handled evidence and when
- **Parallel action**: Start containment while collecting evidence — don't wait
- **Communication**: Notify stakeholders early — legal, PR, and management need to be involved
- **Scope creep**: Incident scope can expand — re-evaluate containment boundaries regularly

## Verification
- Evidence collected and stored on separate forensic system
- Timeline covers all attacker actions from initial access to detection
- All IOCs documented and shared with threat intelligence teams
- Affected systems restored from known-good backups
- Post-incident review completed within 2 weeks
- Remediation actions tracked and verified
- Detection improved to catch similar attacks earlier