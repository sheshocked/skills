---
name: email-deliverability
description: 
category: devops
tags: [email-deliverability]
---

## When to Use
Configure email servers and DNS for high deliverability: SPF, DKIM, DMARC setup, SMTP configuration, IP warming, reputation management, and troubleshooting bounce/spam issues.

## Core Concepts
- **SPF**: DNS record listing authorized sending servers
- **DKIM**: Cryptographic email signing to prove authenticity
- **DMARC**: Policy for handling failed SPF/DKIM checks
- **SMTP relay**: External service (SES, SendGrid, Mailgun) for delivery
- **IP warming**: Gradually increasing volume to build sender reputation
- **Bounce management**: Hard bounces (invalid) vs soft bounces (temporary)

## Workflow
1. Set up SPF record with all authorized senders
2. Generate and publish DKIM keys
3. Configure DMARC policy (start with `p=none`, move to `p=reject`)
4. Set up SMTP relay with proper authentication
5. Monitor deliverability and reputation
6. Warm up new sending IPs gradually

## Key Patterns
```bash
# DNS records for email deliverability
# SPF — authorize sending servers
example.com.  IN TXT "v=spf1 mx a ip4:1.2.3.4 include:amazonses.com -all"

# DKIM — generate key pair
opendkim-genkey -b 2048 -d example.com -s selector1 -v
# Publish public key in DNS:
selector1._domainkey.example.com. IN TXT "v=DKIM1; k=rsa; p=MIIBIjANBgkqhki..."

# DMARC — authentication policy
_dmarc.example.com. IN TXT "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com; ruf=mailto:forensics@example.com; pct=100; adkim=r; aspf=r"

# MX records — mail exchange
example.com. IN MX 10 mail1.example.com.
example.com. IN MX 20 mail2.example.com.

# Reverse DNS (PTR) — set at hosting provider
# 4.3.2.1.in-addr.arpa → mail.example.com
```

```bash
# Postfix SMTP server configuration
# /etc/postfix/main.cf
myhostname = mail.example.com
mydomain = example.com
myorigin = $mydomain
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
relay_domains =
home_mailbox = Maildir/

# TLS
smtpd_use_tls = yes
smtpd_tls_cert_file = /etc/letsencrypt/live/mail.example.com/fullchain.pem
smtpd_tls_key_file = /etc/letsencrypt/live/mail.example.com/privkey.pem
smtpd_tls_security_level = may

# Authentication (Dovecot SASL)
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_auth_enable = yes

# Rate limiting
smtpd_client_connection_rate_limit = 10
smtpd_client_message_rate_limit = 30
```

```python
# Python email sending with proper headers
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import dkim

def send_email(to_addr, subject, body):
    msg = MIMEMultipart()
    msg['From'] = 'noreply@example.com'
    msg['To'] = to_addr
    msg['Subject'] = subject
    msg['List-Unsubscribe'] = '<mailto:unsubscribe@example.com>'

    msg.attach(MIMEText(body, 'html'))

    # Sign with DKIM
    with open('/etc/opendkim/keys/example.com/private.pem', 'rb') as f:
        private_key = f.read()

    sig = dkim.sign(
        msg.as_bytes(),
        selector='selector1',
        domain='example.com',
        privkey=private_key,
    )
    msg['DKIM-Signature'] = sig.decode()

    with smtplib.SMTP('localhost', 587) as server:
        server.starttls()
        server.send_message(msg)
```

```bash
# Email deliverability testing
# Check SPF
dig example.com TXT | grep spf

# Check DKIM
dig selector1._domainkey.example.com TXT

# Check DMARC
dig _dmarc.example.com TXT

# Test with mail-tester.com
# Send test email and get score (aim for 9/10+)

# Check blacklists
# https://mxtoolbox.com/blacklists.aspx
# https://multirbl.valli.org/
```

## Pitfalls
- **SPF DNS lookup limit**: Max 10 DNS lookups in SPF record — use flattening
- **DKIM key length**: Use 2048-bit keys minimum
- **DMARC policy**: Start with `p=none` to monitor, then tighten to `p=quarantine`, then `p=reject`
- **IP warming**: Don't send 10k emails on day one — start with 100/day
- **Feedback loops**: Register with major ISPs for complaint notifications
- **Unsubscribe**: Always include List-Unsubscribe header (CAN-SPAM requirement)

## Verification
```bash
# Full email authentication check
echo "Test email" | mail -s "Test" test@mail-tester.com

# Check blacklists
for bl in zen.spamhaus.org b.barracudacentral.org; do
  echo "=== $bl ==="
  dig +short 4.3.2.1.$bl
done

# Verify DKIM signature
opendkim-testkey -d example.com -s selector1 -vvv
```