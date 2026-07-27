---
name: privacy-anonymity-ops
description: 
category: security
tags: [privacy-anonymity-ops]
---

## When to Use
Use this skill when implementing operational privacy and anonymity — Tor usage, metadata minimization, compartmentation, OPSEC practices, and privacy-preserving communications.

## Core Concepts
- **Metadata minimization**: Reduce identifying information in communications
- **Compartmentation**: Separate identities for different activities
- **Threat modeling**: Identify who you're hiding from and what resources they have
- **Tor routing**: Multi-layered encryption through volunteer relays
- **Tails OS**: Amnesic live operating system for sensitive operations
- **OPSEC**: Operational security — habits that protect identity

## Workflow
1. **Threat model**: Define adversaries and their capabilities
2. **Identity compartmentation**: Separate personas with no overlap
3. **Communication security**: Use encrypted, metadata-minimizing channels
4. **Browsing privacy**: Tor, VPN chains, browser fingerprinting reduction
5. **Device security**: Full-disk encryption, secure boot, tamper evidence
6. **Metadata hygiene**: Strip EXIF, minimize phone metadata, avoid correlation
7. **Physical OPSEC**: Travel security, device border crossing procedures

## Key Patterns

### Tor Configuration
```bash
# Install and configure Tor
apt install tor
systemctl enable --now tor

# /etc/tor/torrc — hardened configuration
SocksPort 9050
SocksPolicy accept 127.0.0.1
SocksPolicy accept ::1
DataDirectory /var/lib/tor
RunAsDaemon 1

# Use bridges for censorship-resistant access
UseBridges 1
Bridge obfs4 192.95.36.142:443 CERT=... IAT_MODE=0
Bridge obfs4 185.177.207.3:80 CERT=... IAT_MODE=0

# Force circuits through specific countries
ExitNodes {us},{ca},{de}
StrictNodes 0

# Never allow local network connections
ExcludeExitNodes {cn},{ru},{ir}

# Verify Tor is working
curl --socks5-hostname 127.0.0.1:9050 https://check.torproject.org/api/ip
# Should return: {"IsTor":true,"IP":"..."}
```

### Browser Fingerprinting Reduction
```bash
# Firefox privacy configuration
# about:config

# Disable WebRTC (leaks real IP)
media.peerconnection.enabled = false
media.peerconnection.ice.default_address_only = true

# Resist fingerprinting
privacy.resistFingerprinting = true
privacy.trackingprotection.enabled = true
privacy.firstparty.isolate = true

# Disable telemetry
toolkit.telemetry.enabled = false
datareporting.healthreport.uploadEnabled = false
browser.ping-centre.telemetry = false

# Disable prefetching
network.prefetch-next = false
network.dns.disablePrefetch = true

# Canvas fingerprint protection
privacy.resistFingerprinting = true  # Returns blank canvas

# Install privacy extensions
# uBlock Origin — ad/tracker blocking
# HTTPS Everywhere — force HTTPS
# Cookie AutoDelete — auto-delete cookies
# Decentraleyes — local CDN emulation
```

### Metadata Stripping
```bash
# Remove EXIF from images
apt install exiftool

# Strip all metadata from image
exiftool -all= image.jpg
exiftool -all= -overwrite_original image.jpg

# Batch strip
find . -name "*.jpg" -exec exiftool -all= -overwrite_original {} \;

# Remove metadata from PDF
exiftool -all= document.pdf

# Remove metadata from video
ffmpeg -i input.mp4 -map_metadata -1 -c:v copy -c:a copy output.mp4

# Remove metadata from office documents
pip install python-docx
python3 -c "
from docx import Document
doc = Document('document.docx')
doc.core_properties.author = ''
doc.core_properties.last_modified_by = ''
doc.core_properties.created = None
doc.core_properties.modified = None
doc.save('clean.docx')
"

# Check what metadata exists
exiftool -a -G1 image.jpg
```

### Communication Security
```bash
# Signal — end-to-end encrypted messaging
# Use Signal Desktop with disappearing messages enabled
# Disable phone number lookup: Settings → Privacy → Phone Number → Nobody

# Matrix/Element — decentralized encrypted messaging
# Self-hosted Matrix server for organizational communication
docker run -d --name matrix-synapse \
  -p 8008:8008 \
  -v /data/synapse:/data \
  matrixdotorg/synapse

# Wire — E2E encrypted with no phone number required
# Create account with email only

# Email encryption with PGP
# Generate key pair
gpg --full-generate-key
gpg --export --armor user@email.com > public.asc
gpg --export-secret-keys user@email.com > private.asc

# Encrypt email
echo "Secret message" | gpg --encrypt --recipient user@email.com

# Decrypt
gpg --decrypt message.gpg

# Verify signature
gpg --verify message.gpg
```

### Identity Compartmentation
```bash
# Create separate VMs for different identities
# Identity 1: Personal
qemu-img create -f qcow2 personal.qcow2 50G
virt-install --name personal --memory 4096 --vcpus 2 \
  --disk path=personal.qcow2 --cdrom tails.iso

# Identity 2: Professional
qemu-img create -f qcow2 professional.qcow2 50G
virt-install --name professional --memory 4096 --vcpus 2 \
  --disk path=professional.qcow2 --cdrom ubuntu.iso

# Never mix identities:
# - Different browsers/profiles
# - Different email addresses
# - Different VPN connections
# - Different physical locations
# - Different writing styles

# DNS leak prevention
# Configure DNS through Tor
cat >> /etc/tor/torrc << 'EOF'
DNSPort 5353
AutomapHostsOnResolve 1
EOF

# Configure resolv.conf
echo "nameserver 127.0.0.1" > /etc/resolv.conf

# Verify no DNS leaks
# Visit: https://dnsleaktest.com
```

### Tails OS Usage
```bash
# Download and verify Tails
wget https://tails.net/tails-amd64-5.8.img
wget https://tails.net/tails-amd64-5.8.img.sig
gpg --keyserver hkps://keys.openpgp.org --recv-keys 0x90127F8B7E7C176DC9D9C620464CA87351752923
gpg --verify tails-amd64-5.8.img.sig tails-amd64-5.8.img

# Create bootable USB
dd if=tails-amd64-5.8.img of=/dev/sdb bs=4M status=progress

# Boot from USB and verify:
# - Tor connection works
# - All traffic routed through Tor
# - Persistent storage encrypted (if using)
# - MAC address spoofed
```

### VPN + Tor Chain
```bash
# VPN → Tor → Destination
# Encrypts traffic from ISP to Tor entry, hides VPN usage from Tor

# Configure
# 1. Connect to VPN first
openvpn --config vpn-config.ovpn

# 2. Then route through Tor
# Firefox → Proxy → SOCKS5 → 127.0.0.1:9050

# Verify chain
curl --socks5-hostname 127.0.0.1:9050 https://ifconfig.me
# Should show Tor exit node IP, not VPN IP
```

## Pitfalls
- **Tor is not a VPN**: Tor exit nodes can see traffic — use HTTPS always
- **Browser fingerprint**: Even with Tor, browser fingerprinting can identify you — use Tails
- **Timing correlation**: Adversaries can correlate entry/exit timing — minimize predictable patterns
- **Phone metadata**: SMS and calls are metadata-rich — prefer Signal
- **Physical security**: Digital privacy is useless if devices are seized — use full-disk encryption
- **Social media**: Posts, photos, and interactions create metadata — minimize exposure

## Verification
- `curl --socks5-hostname 127.0.0.1:9050 https://check.torproject.org/api/ip` — IsTor:true
- No DNS leaks on dnsleaktest.com
- Browser fingerprint on amiunique.org shows average/tor-like profile
- EXIF data stripped from all shared images
- Different identities have zero overlapping metadata
- Full-disk encryption verified (LUKS)
- MAC address randomized on wireless interfaces