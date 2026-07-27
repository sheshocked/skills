---
name: physical-security-ops
description: 
category: security
tags: [physical-security-ops]
---

## When to Use
Use this skill when implementing physical and device security controls — full-disk encryption, secure disposal, tamper evidence, travel opsec, and device hardening.

## Core Concepts
- **Full-disk encryption**: Encrypt entire disk to protect data at rest
- **Secure disposal**: Proper destruction of storage media
- **Tamper evidence**: Detect physical access to devices
- **Travel opsec**: Protect devices during border crossings and travel
- **Device hardening**: BIOS/UEFI security, secure boot, TPM usage
- **Anti-forensics**: Understand what forensic artifacts exist

## Workflow
1. **Encrypt devices**: Full-disk encryption on all laptops and servers
2. **Secure boot**: Enable UEFI Secure Boot, set BIOS password
3. **Tamper detection**: Use tamper-evident seals on critical hardware
4. **Travel preparation**: Minimize data carried, use travel-specific devices
5. **Secure disposal**: Certified destruction of decommissioned hardware
6. **Lost device protocol**: Remote wipe, credential rotation, incident report
7. **Physical access controls**: Badge access, camera monitoring, clean desk

## Key Patterns

### Full-Disk Encryption (Linux)
```bash
# LUKS full-disk encryption (clean install)
# During Ubuntu/Kali installation, select "Encrypt the new installation"

# Manual LUKS setup on existing disk (DESTRUCTIVE — back up first)
# Partition setup
cryptsetup luksFormat /dev/sda2
cryptsetup open /dev/sda2 cryptroot
mkfs.ext4 /dev/mapper/cryptroot
mount /dev/mapper/cryptroot /mnt

# LUKS header backup (CRITICAL — without this, data is unrecoverable)
cryptsetup luksHeaderBackup /dev/sda2 --header-backup-file /external/luks-header-backup.img
chmod 600 /external/luks-header-backup.img

# Add additional key for recovery
dd if=/dev/urandom of=/external/recovery-key.key bs=1 count=256
chmod 400 /external/recovery-key.key
cryptsetup luksAddKey /dev/sda2 /external/recovery-key.key

# Verify encryption
cryptsetup isLuks /dev/sda2 && echo "Encrypted" || echo "Not encrypted"
cryptsetup luksDump /dev/sda2

# Auto-unlock with TPM (systemd-cryptenroll)
systemd-cryptenroll /dev/sda2 --tpm2-device=auto
```

### Device Hardening (UEFI/BIOS)
```bash
# Set BIOS/UEFI password (varies by manufacturer)
# Usually done through BIOS setup interface at boot

# Enable Secure Boot (UEFI)
# Most modern distros support this — enable in BIOS

# Disable boot from USB/CD (prevent live boot attacks)
# Set boot order to internal disk only

# Enable TPM 2.0
# Used for: BitLocker, LUKS auto-unlock, measured boot

# Disable Thunderbolt/DMA ports when not in use
#防止 DMA attacks via Thunderbolt
echo '0' > /sys/bus/thunderbolt/devices/0-0/authorized

# Disable USB storage (if not needed)
echo 'install usb-storage /bin/true' > /etc/modprobe.d/disable-usb-storage.conf
```

### Secure Disposal
```bash
# Software wipe (for functional drives)
# DoD 5220.22-M (3-pass)
dd if=/dev/urandom of=/dev/sda bs=1M status=progress
dd if=/dev/urandom of=/dev/sda bs=1M status=progress
dd if=/dev/urandom of=/dev/sda bs=1M status=progress

# ATA Secure Erase (hardware-level)
sudo hdparm --user-master u --security-set-pass p /dev/sda
sudo hdparm --security-erase p /dev/sda

# SSD TRIM/discard (may not be sufficient alone)
blkdiscard /dev/sdb

# Verify wipe
hexdump -C /dev/sda | head -20
# Should show all zeros or random data

# Physical destruction (for highly sensitive data)
# Use certified destruction service or:
# Drill through platters
# Degauss (HDD only — doesn't work on SSD)
# Industrial shredder
# Record destruction with certificate

# Document disposal
echo "$(date): Destroyed disk $SERIAL_NUMBER, capacity $SIZE" >> /secure/disposal-log.txt
```

### Tamper Evidence
```bash
# Physical tamper-evident seals
# Use numbered, tamper-evident stickers on:
# - Server cases
# - Network equipment
# - USB ports on servers
# - BIOS/UEFI access panels

# Digital tamper detection
# File integrity monitoring with AIDE
apt install aide
aideinit
aide --check

# Monitor critical files
# /etc/aide/aide.conf
# Monitor: /boot, /etc, /usr/bin, /usr/sbin, /lib

# Schedule daily checks
cat > /etc/cron.daily/aide-check << 'EOF'
#!/bin/bash
aide --check | mail -s "AIDE Integrity Check" security@example.com
EOF
chmod +x /etc/cron.daily/aide-check

# TPM-based measured boot
# Verify boot chain integrity
tpm2_pcrread sha256:0,1,2,3,4,5,7
```

### Travel Opsec
```bash
# Pre-travel checklist
# 1. Back up all data to secure location
# 2. Encrypt all devices (verify LUKS/BitLocker)
# 3. Create travel-specific device with minimal data
# 4. Disable unnecessary services (Bluetooth, WiFi)
# 5. Set strong PINs/passwords on all devices
# 6. Enable remote wipe capability

# Create travel device
# Minimal Tails USB with no persistent storage
# Only carry necessary data — leave sensitive data at home

# Border crossing preparation
# - Power off devices before crossing (no hibernation)
# - Be prepared to provide device passwords if asked
# - Know local laws about device searches
# - Consider leaving devices at home if not needed

# WiFi security while traveling
# Always use VPN on public WiFi
# Prefer mobile hotspot over public WiFi
# Disable auto-connect to WiFi networks
# Use WPA3 where available

# After travel
# Change all passwords that were used while traveling
# Check for unauthorized software or modifications
# Verify file integrity with AIDE
```

### Lost Device Protocol
```bash
#!/bin/bash
# Lost device response checklist

echo "=== LOST DEVICE RESPONSE ==="

# 1. Remote wipe (Android)
# Use Google Find My Device: https://www.google.com/android/find
# Use Samsung Find My Mobile: https://findmymobile.samsung.com

# 2. Remote wipe (iOS)
# Use iCloud Find My iPhone: https://www.icloud.com/find

# 3. Remote wipe (Linux — pre-configured with cryptsetup)
# If using LUKS with network-bound unlock:
# Revoke the key server certificate

# 4. Rotate credentials
echo "Rotate these credentials:"
echo "  - Email password"
echo "  - VPN credentials"
echo "  - SSH keys (revoke old, generate new)"
echo "  - Cloud provider access keys"
echo "  - 2FA tokens (re-enroll)"

# 5. Notify security team
echo "Notify: security@example.com"

# 6. File incident report
echo "Document: device type, serial number, last known location"
echo "Document: data that may have been on device"
```

## Pitfalls
- **Encryption without backup**: If you lose the encryption key, data is permanently lost — backup LUKS headers
- **SSD TRIM limitations**: Software wipe on SSDs may not reach all blocks — use manufacturer secure erase
- **Cold boot attacks**: RAM contents persist after power off — use Tails or power off completely before travel
- **TPM vulnerabilities**: TPM can be attacked with physical access — not a complete security solution
- **Border searches**: In some countries, refusing to provide passwords may result in device seizure
- **Remote wipe timing**: Remote wipe requires internet connection — if device is offline, wipe won't execute

## Verification
- `cryptsetup isLuks /dev/sda` — all disks encrypted
- `secureboot --verify` — Secure Boot enabled
- AIDE check passes with no unauthorized changes
- LUKS header backed up to secure external location
- Remote wipe capability tested and working
- Travel device has minimal data
- All devices powered off before border crossing
- Disposal log shows all decommissioned media properly destroyed