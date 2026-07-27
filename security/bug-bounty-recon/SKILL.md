---
name: bug-bounty-recon
description: 
category: security
tags: [bug-bounty-recon]
---

## When to Use
Use this skill when conducting reconnaissance for bug bounty programs — subdomain enumeration, asset discovery, fingerprinting, technology detection, and scope management to maximize coverage efficiently.

## Core Concepts
- **Passive recon**: OSINT gathering without touching the target (WHOIS, DNS, certificates)
- **Active recon**: Direct probing of discovered assets (port scanning, tech fingerprinting)
- **Asset deduplication**: Normalize and deduplicate discovered subdomains
- **Scope management**: Stay within program scope — flag out-of-scope assets
- **Tool chaining**: Pipe outputs between tools for comprehensive coverage
- **Automation**: Build repeatable recon pipelines for efficiency

## Workflow
1. **Seed enumeration**: Start with target domain + program scope
2. **Subdomain brute-force**: Use multiple tools and wordlists
3. **Certificate transparency**: Search CT logs for historical subdomains
4. **DNS resolution**: Resolve all subdomains, identify live hosts
5. **Port scanning**: Scan live hosts for open services
6. **Technology fingerprinting**: Identify frameworks, versions, services
7. **Parameter discovery**: Find API endpoints and hidden parameters
8. **Dedup & organize**: Merge results, tag by scope, prioritize by interestingness

## Key Patterns

### Subdomain Enumeration Pipeline
```bash
# 1. Amass — comprehensive enumeration
amass enum -passive -d target.com -o amass_passive.txt
amass enum -active -d target.com -brute -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -o amass_active.txt

# 2. Subfinder — fast passive enumeration
subfinder -d target.com -all -o subfinder.txt

# 3. crt.sh — Certificate Transparency logs
curl -s "https://crt.sh/?q=%.target.com&output=json" | jq -r '.[].name_value' | sort -u > crtsh.txt

# 4. dnsxbrute — DNS brute-force
dnsxbrute -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -o dnsxbrute.txt

# Combine and deduplicate
cat amass_passive.txt amass_active.txt subfinder.txt crtsh.txt dnsxbrute.txt | sort -u > all_subdomains.txt
```

### Live Host Discovery
```bash
# httpx — probe for live HTTP services
cat all_subdomains.txt | httpx -silent -status-code -title -tech-detect -follow-redirects -o live_hosts.txt

# Alternative: use dnsx for DNS resolution
cat all_subdomains.txt | dnsx -silent -a -resp-only > resolved_ips.txt

# Masscan for fast port scanning
cat resolved_ips.txt | masscan -p1-65535 --rate=10000 -oJ masscan.json

# nuclei — vulnerability template scanning
cat live_hosts.txt | nuclei -severity critical,high,medium -o nuclei_results.txt
```

### Technology Fingerprinting
```bash
# WhatWeb — technology identification
whatweb -a 3 target.com

# Wappalyzer CLI alternative
cat live_hosts.txt | while read url; do
  echo "=== $url ==="
  curl -sI "$url" | grep -iE '(server|x-powered|x-aspnet|x-generator)'
done

# JS file analysis — find API endpoints and secrets
cat live_hosts.txt | gau | grep '\.js$' | sort -u > js_files.txt
cat js_files.txt | while read js; do
  echo "=== $js ==="
  curl -s "$js" | grep -oE '(https?://[^"'"'"']+|/[a-zA-Z0-9/_-]+/[a-zA-Z0-9/_-]+)' | head -20
done

# Check for common files
for f in robots.txt sitemap.xml .git/config .env .DS_Store; do
  curl -sI "https://target.com/$f" | head -1
done
```

### Parameter Discovery
```bash
# Arjun — parameter discovery
arjun -u "https://target.com/api/endpoint" -m GET POST JSON

# ParamSpider — mine parameters from JS
paramspider -d target.com -o params.txt

# GAU — gather all URLs
echo "target.com" | gau --blacklist png,jpg,gif,css,woff | sort -u > urls.txt

# Extract parameters from URLs
cat urls.txt | grep '?' | sed 's/.*?//' | tr '&' '\n' | cut -d= -f1 | sort -u
```

### Scope Management Script
```bash
#!/bin/bash
# scope_check.sh — verify targets are in scope
SCOPE="target.com *.target.com"
while read subdomain; do
  match=false
  for pattern in $SCOPE; do
    if [[ "$subdomain" == $pattern ]] || [[ "$subdomain" == *."${pattern#*.}" ]]; then
      match=true
      break
    fi
  done
  if $match; then
    echo "[IN-SCOPE] $subdomain"
  else
    echo "[OUT-SCOPE] $subdomain" >> out_of_scope.txt
  fi
done < all_subdomains.txt

echo "Out-of-scope assets saved to out_of_scope.txt"
```

## Pitfalls
- **Scope violations**: Always check program scope before testing — out-of-scope testing = potential legal issues
- **Rate limiting**: Aggressive scanning triggers IP bans — use slow rates and VPN rotation
- **Tool output noise**: Most tools produce false positives — validate manually
- **Subdomain takeover**: Check for dangling CNAME records but verify before claiming
- **Duplicated effort**: Deduplicate across tools — same subdomain from amass and subfinder is one asset
- **Storage management**: Recon generates massive output files — clean up regularly

## Verification
- Subdomain count > 1000 for large programs
- All live hosts probed with httpx
- Technology stack identified for top 50 live hosts
- No out-of-scope assets targeted
- Results organized by priority and technology
- Key findings (admin panels, API endpoints, staging environments) highlighted
- All recon data backed up before testing begins