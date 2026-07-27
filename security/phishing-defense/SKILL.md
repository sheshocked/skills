---
name: phishing-defense
description: 
category: security
tags: [phishing-defense]
---

## When to Use
Use this skill when implementing anti-phishing controls — SPF/DKIM/DMARC email authentication, phishing simulation campaigns, user awareness training, and link/attachment analysis.

## Core Concepts
- **SPF**: Sender Policy Framework — specifies which IPs can send email for your domain
- **DKIM**: DomainKeys Identified Mail — cryptographic signature on email headers
- **DMARC**: Domain-based Message Authentication — policy for handling SPF/DKIM failures
- **Phishing simulation**: Controlled tests to measure user susceptibility
- **Link analysis**: URL parsing, redirect chain analysis, domain reputation
- **Reporting**: Internal phishing report process for suspicious emails

## Workflow
1. **Email authentication**: Configure SPF, DKIM, and DMARC for your domain
2. **Anti-spoofing**: Block emails that fail authentication
3. **User training**: Regular phishing awareness training
4. **Simulation campaigns**: Send simulated phishing emails, track click rates
5. **Reporting mechanism**: Easy "report phishing" button in email clients
6. **Link analysis**: Set up tools to analyze suspicious URLs
7. **DMARC monitoring**: Track authentication failures and spoofing attempts
8. **Continuous improvement**: Iterate based on simulation results

## Key Patterns

### SPF Configuration
```dns
; DNS TXT record for SPF
; Allows specific IPs and Google Workspace to send email
example.com.  IN TXT  "v=spf1 include:_spf.google.com include:mailgun.org ip4:203.0.113.0/24 -all"
```

### DKIM Configuration
```bash
# Generate DKIM key pair
opendkim-genkey -s default -d example.com -b 2048
# Creates: default.private (private key) and default.txt (DNS record)

# DNS TXT record for DKIM
default._domainkey.example.com. IN TXT  "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA..."

# Configure Postfix to sign outgoing mail
# /etc/opendkim.conf
Canonicalization    relaxed/simple
Mode                sv
SubDomains          yes
Syslog              yes
KeyTable            refile:/etc/opendkim/KeyTable
SigningTable        refile:/etc/opendkim/SigningTable

# /etc/opendkim/KeyTable
default._domainkey.example.com  example.com:default:/etc/opendkim/keys/example.com/default.private

# /etc/opendkim/SigningTable
*@example.com  default._domainkey.example.com
```

### DMARC Configuration
```dns
; DMARC policy — start with monitoring, then enforce
_dmarc.example.com.  IN TXT  "v=DMARC1; p=none; rua=mailto:dmarc@example.com; ruf=mailto:dmarc-forensics@example.com; fo=1; adkim=s; aspf=s; pct=100"

; After monitoring period (2-4 weeks), move to quarantine
_dmarc.example.com.  IN TXT  "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com; ruf=mailto:dmarc-forensics@example.com; fo=1; adkim=s; aspf=s; pct=100"

; Final: reject unauthenticated email
_dmarc.example.com.  IN TXT  "v=DMARC1; p=reject; rua=mailto:dmarc@example.com; ruf=mailto:dmarc-forensics@example.com; fo=1; adkim=s; aspf=s; pct=100"
```

### Phishing Simulation Campaign
```python
#!/usr/bin/env python3
# Phishing simulation campaign manager
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import random
import uuid
from datetime import datetime

def send_phishing_simulation(
    smtp_server: str,
    smtp_port: int,
    from_email: str,
    target_emails: list[str],
    template: str
):
    # Send simulated phishing emails for awareness training
    campaign_id = str(uuid.uuid4())[:8]
    results = []

    with smtplib.SMTP(smtp_server, smtp_port) as server:
        server.starttls()

        for email in target_emails:
            msg = MIMEMultipart('alternative')
            msg['From'] = from_email
            msg['To'] = email
            msg['Subject'] = "URGENT: Password Reset Required"

            # Track unique link per user
            tracking_id = f"{campaign_id}-{email.split('@')[0]}"
            tracking_url = f"https://training.internal.com/landing/{tracking_id}"

            html = (
                '<html><body>'
                '<p>Dear Employee,</p>'
                '<p>Your password will expire in 24 hours. Please click below to reset:</p>'
                f'<a href="{tracking_url}" style="background:#0066cc;color:white;padding:10px 20px;text-decoration:none;border-radius:5px;">Reset Password</a>'
                '<p>If you did not request this, please contact IT Security.</p>'
                '</body></html>'
            )

            msg.attach(MIMEText(html, 'html'))
            server.sendmail(from_email, email, msg.as_string())

            results.append({
                "campaign": campaign_id,
                "email": email,
                "tracking_id": tracking_id,
                "sent_at": datetime.utcnow().isoformat()
            })

    return results

def analyze_results(campaign_id: str, db_connection):
    # Analyze phishing simulation results
    import sqlite3

    # Count total sent
    total = db_connection.execute(
        "SELECT COUNT(*) FROM campaign_results WHERE campaign = ?",
        (campaign_id,)
    ).fetchone()[0]

    # Count clicks
    clicked = db_connection.execute(
        "SELECT COUNT(*) FROM click_events WHERE campaign = ?",
        (campaign_id,)
    ).fetchone()[0]

    # Count reports
    reported = db_connection.execute(
        "SELECT COUNT(*) FROM report_events WHERE campaign = ?",
        (campaign_id,)
    ).fetchone()[0]

    click_rate = (clicked / total * 100) if total > 0 else 0
    report_rate = (reported / total * 100) if total > 0 else 0

    print(f"=== Campaign {campaign_id} Results ===")
    print(f"Total sent:     {total}")
    print(f"Clicked:        {clicked} ({click_rate:.1f}%)")
    print(f"Reported:       {reported} ({report_rate:.1f}%)")
    print(f"Click rate target: <5%")
    print(f"Report rate target: >30%")

    return {
        "total": total,
        "clicked": clicked,
        "reported": reported,
        "click_rate": click_rate,
        "report_rate": report_rate
    }
```

### Link Analysis Tool
```python
#!/usr/bin/env python3
# Analyze suspicious URLs for phishing indicators
import re
import requests
from urllib.parse import urlparse, parse_qs
import hashlib

def analyze_url(url: str) -> dict:
    # Analyze a URL for phishing indicators
    findings = []

    parsed = urlparse(url)

    # Check for IP address instead of domain
    if re.match(r'^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$', parsed.hostname):
        findings.append("CRITICAL: IP address used instead of domain name")

    # Check for suspicious TLD
    suspicious_tlds = ['.tk', '.ml', '.ga', '.cf', '.gq', '.xyz', '.top', '.buzz']
    if any(parsed.hostname.endswith(tld) for tld in suspicious_tlds):
        findings.append("WARNING: Suspicious TLD")

    # Check for URL shorteners
    shorteners = ['bit.ly', 'tinyurl.com', 'goo.gl', 't.co', 'is.gd', 'v.gd']
    if parsed.hostname in shorteners:
        findings.append("INFO: URL shortener detected — resolve before clicking")

    # Check for @ symbol (credential theft)
    if '@' in url.split('//')[1]:
        findings.append("CRITICAL: @ symbol in URL — potential credential theft")

    # Check for excessive redirects
    try:
        resp = requests.head(url, allow_redirects=True, timeout=10)
        redirect_chain = resp.history
        if len(redirect_chain) > 3:
            findings.append(f"WARNING: {len(redirect_chain)} redirects detected")
    except requests.RequestException:
        findings.append("WARNING: Could not follow redirects")

    # Check domain age (if available via API)
    # Check against phishing databases
    # Check SSL certificate details

    return {
        "url": url,
        "domain": parsed.hostname,
        "findings": findings,
        "risk_level": "HIGH" if any("CRITICAL" in f for f in findings) else
                      "MEDIUM" if any("WARNING" in f for f in findings) else "LOW"
    }

if __name__ == "__main__":
    import sys
    result = analyze_url(sys.argv[1])
    print(f"Risk: {result['risk_level']}")
    for finding in result['findings']:
        print(f"  - {finding}")
```

### Email Authentication Verification
```bash
# Check SPF record
dig TXT example.com | grep "spf1"
# Expected: v=spf1 include:_spf.google.com -all

# Check DKIM record
dig TXT default._domainkey.example.com
# Expected: v=DKIM1; k=rsa; p=...

# Check DMARC record
dig TXT _dmarc.example.com
# Expected: v=DMARC1; p=reject; rua=mailto:...

# Send test email and verify authentication
# Use: https://www.mail-tester.com/
# Send email to the test address, then check score

# Check email headers for authentication results
# Look for: SPF=PASS, DKIM=PASS, DMARC=PASS

# Monitor DMARC reports
# Aggregate reports sent to rua email weekly
# Analyze with: https://dmarcian.com/ or https://mxtoolbox.com/dmarc.aspx
```

## Pitfalls
- **SPF include limits**: DNS lookup limit of 10 — chained includes can exceed this
- **DKIM key rotation**: Rotate DKIM keys periodically — compromise of private key allows spoofing
- **DMARC policy too strict too fast**: Start with p=none, monitor, then move to quarantine/reject
- **Subdomain spoofing**: DMARC doesn't protect subdomains by default — add `sp=reject` for subdomains
- **User fatigue**: Too many phishing simulations cause annoyance — balance frequency
- **False positives**: Legitimate emails may fail authentication — monitor DMARC reports

## Verification
- `dig TXT example.com` shows valid SPF record
- `dig TXT default._domainkey.example.com` shows valid DKIM record
- `dig TXT _dmarc.example.com` shows valid DMARC record
- mail-tester.com score > 9/10
- Phishing simulation click rate < 5%
- Phishing report rate > 30%
- No spoofed emails from your domain in DMARC reports
- All employees completed phishing awareness training