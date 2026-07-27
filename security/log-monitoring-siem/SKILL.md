---
name: log-monitoring-siem
description: 
category: security
tags: [log-monitoring-siem]
---

## When to Use
Use this skill when setting up centralized log monitoring, detection rules, alerting pipelines, and threat hunting queries — using tools like Wazuh, ELK, Splunk, or custom SIEM solutions.

## Core Concepts
- **Centralized logging**: All logs aggregated to single platform for correlation
- **Detection rules**: Sigma/YARA-like rules to identify suspicious patterns
- **Alert pipelines**: Enrich, correlate, and route alerts to appropriate responders
- **Threat hunting**: Proactive queries to find undetected threats
- **Log sources**: Authentication, network, application, endpoint, cloud audit logs
- **Retention**: Balance storage costs with compliance requirements

## Workflow
1. **Log source inventory**: Identify all log sources (servers, apps, cloud services)
2. **Agent deployment**: Install log shippers (Filebeat, Fluentd, Wazuh agent)
3. **Parsing/normalization**: Parse logs into structured format
4. **Detection rules**: Create rules for known attack patterns
5. **Alert tuning**: Reduce false positives, set appropriate thresholds
6. **Dashboard creation**: Build operational dashboards for key metrics
7. **Threat hunting**: Run scheduled queries to find hidden threats
8. **Response automation**: Auto-respond to high-confidence alerts

## Key Patterns

### Wazuh Setup
```bash
# Install Wazuh manager
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
bash ./wazuh-install.sh -a

# Install agent on endpoint
curl -sO https://packages.wazuh.com/4.7/wazuh-agent-4.7.0-1.x86_64.rpm
rpm -ihv wazuh-agent-4.7.0-1.x86_64.rpm

# Configure agent
sed -i 's/<address>MANAGER_IP<\/address>/<address>10.0.0.5<\/address>/' /var/ossec/etc/ossec.conf
systemctl restart wazuh-agent

# Custom detection rule
cat > /var/ossec/etc/rules/local_rules.xml << 'EOF'
<group name="custom,">
  <rule id="100001" level="12">
    if_sid 5712
    <field name="net.peer.ip" type="pcre2">^(?!10\.0\.0\.)</field>
    <description>SSH login from external IP</description>
    <mitre>
      <id>T1021.004</id>
    </mitre>
  </rule>

  <rule id="100002" level="10">
    <decoded_as>sshd</decoded_as>
    <action>failed</action>
    <frequency>5</frequency>
    <timeframe>300</timeframe>
    <description>SSH brute force: 5 failures in 5 minutes</description>
  </rule>

  <rule id="100003" level="12">
    <decoded_as>sshd</decoded_as>
    <action>failed</action>
    <frequency>20</frequency>
    <timeframe>600</timeframe>
    <description>SSH brute force: 20 failures in 10 minutes</description>
  </rule>
</group>
EOF
```

### Sigma Rules for Detection
```yaml
# sigma_rules/brute_force_ssh.yml
title: SSH Brute Force
id: 12345678-1234-1234-1234-123456789abc
status: experimental
description: Detects SSH brute force attempts
author: Security Team
date: 2024/01/15
tags:
  - attack.credential_access
  - attack.t1110.001
logsource:
  product: linux
  service: sshd
detection:
  selection:
    event_type: sshd_authentication
    action: failed
  condition: selection | count(src_ip) by src_ip > 10 within 5m
falsepositives:
  - Legitimate password changes
level: high

---
# sigma_rules/suspicious_dns.yml
title: Suspicious DNS Query
id: 87654321-4321-4321-4321-cba987654321
status: experimental
description: Detects DNS queries to known malicious domains
logsource:
  product: linux
  service: dns
detection:
  selection:
    query_type: A
  filter:
    query|endswith:
      - .malware-c2.com
      - .data-exfil.net
  condition: selection and not filter
level: critical
```

### ELK Stack Detection Pipeline
```json
// Logstash pipeline for SSH auth detection
{
  "input": {
    "beats": {
      "port": 5044
    }
  },
  "filter": {
    "grok": {
      "match": {
        "message": "%{SYSLOGTIMESTAMP:timestamp} %{SYSLOGHOST:hostname} sshd\\[%{NUMBER:pid}\\]: %{GREEDYDATA:msg}"
      }
    },
    "if [msg] =~ /Failed password/ {
      "grok": {
        "match": {
          "msg": "Failed password for (invalid user )?%{USER:username} from %{IP:src_ip} port %{NUMBER:src_port}"
        }
      },
      "mutate" => {
        "add_field" => { "event_type" => "ssh_brute_force" }
      }
    },
    "if [msg] =~ /Accepted publickey/ {
      "mutate" => {
        "add_field" => { "event_type" => "ssh_success" }
      }
    }
  },
  "output" {
    "elasticsearch" => {
      "index" => "ssh-auth-%{+YYYY.MM.dd}"
    }
    "if [event_type] == 'ssh_brute_force'" {
      "webhook" => {
        "url" => "https://alerts.internal.com/ssh-brute-force"
      }
    }
  }
}
```

### Threat Hunting Queries
```bash
# ELK/KQL — find unusual outbound connections
# Find processes making connections to new external IPs
event.category:network and destination.ip:* and not destination.ip:(10.0.0.0/8 or 172.16.0.0/12 or 192.168.0.0/16) | stats count by source.process.name, destination.ip, destination.port | where count > 1

# Find unusual scheduled tasks
process.name:"at.exe" or process.name:"schtasks.exe" | stats count by user.name, process.args | where count > 0

# Find PowerShell with encoded commands
process.name:"powershell.exe" and process.args:(*encoded* or *enc* or *FromBase64String*) | table timestamp, user.name, process.args

# Splunk — lateral movement detection
index=windows EventCode=4624 Logon_Type=3 | stats count by src_ip, dest_host | where count > 50

# Wazuh custom query — find deleted log files
SELECT * FROM syscheck WHERE event='deleted' AND file LIKE '/var/log/%';
```

### Alert Dashboard Setup
```bash
# Wazuh API — create custom dashboard
curl -k -X GET "https://wazuh-manager:55000/rules" \
  -H "Authorization: Bearer $TOKEN" | jq '.data.items[] | select(.level >= 10)'

# Key metrics to monitor
# 1. Failed SSH attempts per hour (threshold: >100)
# 2. New listening ports (baseline comparison)
# 3. Process spawning from unusual parents
# 4. DNS queries to uncategorized domains
# 5. File integrity changes in /etc, /usr/bin
# 6. Privilege escalation events
```

## Pitfalls
- **Log volume**: Centralized logging generates massive data — use filtering and sampling
- **Alert fatigue**: Too many low-severity alerts cause responders to ignore real threats
- **Time synchronization**: Logs from different systems must use NTP — otherwise correlation fails
- **Storage costs**: Keep hot data (7 days) fast, warm data (30 days) compressed, cold data (1 year) archived
- **False positives**: Tune rules aggressively — each false positive reduces trust in the system
- **Retention compliance**: Different regulations require different retention periods — document requirements

## Verification
- All critical log sources shipping to SIEM (verify with test events)
- Detection rules tested with known attack samples (Atomic Red Team)
- Alert pipeline delivers to correct teams within SLA
- Dashboards show real-time status of key security metrics
- Threat hunting queries return actionable results monthly
- Log retention meets compliance requirements
- Time synchronization accurate across all log sources