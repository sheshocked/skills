---
name: email-deliverability
description: Configure SPF, DKIM, DMARC records and secure Postfix servers for high deliverability.
category: devops
tags: [email, postfix, spf, dkim, dmarc]
---

# Email Deliverability

## When to Use
Use when deploying applications sending transaction emails (registration, notifications) to prevent spam flags.

## Prerequisites
- Domain name access.

## Workflow
1. Generate DKIM keys and install OpenDKIM.
2. Publish TXT records for SPF, DKIM, and DMARC in DNS panel.
3. Configure Postfix to sign outbounds with OpenDKIM.

## Key Patterns
```txt
# DNS Records
# SPF
v=spf1 ip4:185.71.219.72 -all

# DMARC
_dmarc.mydomain.com TXT "v=DMARC1; p=reject; pct=100; rua=mailto:dmarc@mydomain.com"
```

## Pitfalls
- **Incorrect alignment:** Ensure DKIM signing selector matches target domains exactly.
- **Missing reverse DNS:** Ensure VPS IP has valid rDNS matching mail server hostname.

## Verification
- Send check email to `mail-tester.com` to score deliverability.
- Run `dig txt default._domainkey.mydomain.com` to verify record.
