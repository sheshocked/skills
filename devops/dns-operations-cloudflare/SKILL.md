---
name: dns-operations-cloudflare
description: Automate dynamic DNS updates, Cloudflare records management, and origin certificate renewals.
category: devops
tags: [dns, cloudflare-api, ddns, acme-sh, ssl-cert]
---

# Dns Operations Cloudflare

## When to Use
Use to update DNS records automatically when VPS IP changes (DDNS) and renew SSL certificates via DNS challenge.

## Prerequisites
- Cloudflare API Token.

## Workflow
1. Get current public IP.
2. Trigger Cloudflare API to update A records.
3. Generate SSL certificates using acme.sh script.

## Key Patterns
```bash
# Update record via Cloudflare API
curl -X PUT "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/$RECORD_ID"      -H "Authorization: Bearer $CF_API_TOKEN"      -H "Content-Type: application/json"      --data '{"type":"A","name":"sub.domain.com","content":"'"$CURRENT_IP"'","ttl":120,"proxied":false}'
```

## Pitfalls
- **API Token privileges:** Ensure api token limits permissions only to DNS write access for targeted zones.
- **DNS Propagation lag:** Use low TTL (120s) for DDNS endpoints.

## Verification
- Query DNS: `dig +short sub.domain.com` matches current IP.
- Verify certificates expire dates.
