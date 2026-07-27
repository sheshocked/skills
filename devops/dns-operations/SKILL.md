---
name: dns-operations
description: 
category: devops
tags: [dns-operations]
---

## When to Use
Manage DNS records, zones, and resolution for production services. Covers record types, zone transfers, DNSSEC, split-horizon DNS, and provider management (Cloudflare, Route 53, DigitalOcean).

## Core Concepts
- **Record types**: A, AAAA, CNAME, MX, TXT, SRV, CAA, NS
- **TTL**: Time-To-Live — how long DNS results are cached
- **Split-horizon**: Different responses for internal vs external queries
- **DNSSEC**: Cryptographic signing of DNS records to prevent spoofing
- **GeoDNS**: Route users to nearest endpoint based on location
- **Propagation**: DNS changes take time to spread (propagation delay)

## Workflow
1. Plan DNS architecture (primary/secondary providers, TTL strategy)
2. Set up records with appropriate TTLs
3. Configure failover/health checks
4. Implement DNSSEC if needed
5. Monitor DNS resolution and propagation

## Key Patterns
```bash
# DNS record types — when to use each
# A record     — maps hostname to IPv4
# AAAA record  — maps hostname to IPv6
# CNAME        — alias to another hostname (can't coexist with other records)
# MX           — mail exchange servers
# TXT          — verification, SPF, DKIM, DMARC
# SRV          — service discovery (_service._proto.hostname)
# CAA          — which CAs can issue certificates
# NS           — nameserver delegation
```

```bash
# Cloudflare DNS management via API
# Create DNS record
curl -X POST "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" \
  -H "Authorization: Bearer CF_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data '{
    "type": "A",
    "name": "api.example.com",
    "content": "1.2.3.4",
    "ttl": 300,
    "proxied": true
  }'

# List all records
curl -s "https://api.cloudflare.com/client/v4/zones/ZONE_ID/dns_records" \
  -H "Authorization: Bearer CF_API_TOKEN" | jq '.result[] | {name, type, content, ttl}'
```

```bash
# DNS troubleshooting
dig example.com +short                      # Quick IP lookup
dig example.com ANY +noall +answer          # All records
dig example.com +trace                      # Full resolution path
dig @1.1.1.1 example.com +dnssec           # DNSSEC verification
dig example.com MX +short                   # Mail servers
dig _dmarc.example.com TXT +short           # DMARC record

# Check propagation
for ns in 8.8.8.8 1.1.1.1 208.67.222.222 9.9.9.9; do
  echo "=== $ns ==="
  dig @$ns example.com +short
done

# Verify TTL
dig example.com | grep -A1 "ANSWER SECTION"
```

```bash
# Split-horison DNS (BIND named.conf)
view "internal" {
  match-clients { 10.0.0.0/8; 172.16.0.0/12; };
  zone "example.com" {
    type master;
    file "internal/example.com.zone";
  };
};

view "external" {
  match-clients { any; };
  zone "example.com" {
    type master;
    file "external/example.com.zone";
  };
};
```

```bash
# DMARC + SPF + DKIM DNS records
# SPF — who can send mail for your domain
example.com.  IN TXT "v=spf1 mx a ip4:1.2.3.4 -all"

# DKIM — email signing
selector._domainkey.example.com. IN TXT "v=DKIM1; k=rsa; p=MIIBIjAN..."

# DMARC — policy for failed authentication
_dmarc.example.com. IN TXT "v=DMARC1; p=reject; rua=mailto:dmarc@example.com; pct=100"

# CAA — restrict certificate issuance
example.com. IN CAA 0 issue "letsencrypt.org"
example.com. IN CAA 0 issuewild ";"
```

## Pitfalls
- **CNAME conflicts**: CNAME can't coexist with other records at same name
- **TTL too high**: Changes take longer to propagate (set low during migrations)
- **Propagation delay**: Allow 24-48h for global propagation
- **DNSSEC key rotation**: Plan rotation carefully; misconfiguration breaks resolution
- **MX priority**: Lower number = higher priority; set multiple for redundancy
- **IPv6**: Always add AAAA records alongside A records for dual-stack

## Verification
```bash
# Full DNS audit
dig example.com ANY +noall +answer
dig example.com MX +short
dig _dmarc.example.com TXT +short
dig example.com CAA +short

# Test DMARC policy
# Send test email and check DMARC report
# https://www.mail-tester.com/

# Verify DNSSEC
dig example.com +dnssec +short
```