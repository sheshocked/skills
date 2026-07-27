---
name: tls-certificate-automation
description: 
category: protocols
tags: [tls-certificate-automation]
---

## When to Use
Automate TLS certificates: ACME with certbot/acme.sh, wildcard certs, renewal hooks.

## Certbot Setup
```bash
# Install
apt install certbot

# Obtain certificate
certbot certonly --nginx -d yourdomain.com

# Wildcard (requires DNS challenge)
certbot certonly --manual --preferred-challenges dns -d "*.yourdomain.com"

# Auto-renewal (cron)
0 0 1 * * certbot renew --post-hook "systemctl reload nginx"
```

## acme.sh Setup
```bash
# Install
curl https://get.acme.sh | sh

# Issue cert
acme.sh --issue -d yourdomain.com --webroot /var/www/html

# Install cert
acme.sh --install-cert -d yourdomain.com \
    --key-file /etc/nginx/ssl/key.pem \
    --fullchain-file /etc/nginx/ssl/cert.pem \
    --reloadcmd "systemctl reload nginx"
```

## Pitfalls
- **Rate limits**: Let's Encrypt has rate limits per domain
- **Wildcard**: Only available via DNS challenge
- **Renewal**: Must automate — certs expire every 90 days

## Verification
- Check cert expiry with `openssl x509 -enddate -noout`
- Test SSL with ssllabs.com/ssltest
- Verify auto-renewal works