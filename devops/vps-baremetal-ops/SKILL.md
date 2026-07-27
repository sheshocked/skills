---
name: vps-baremetal-ops
description: 
category: devops
tags: [vps-baremetal-ops]
---

## When to Use
Provision, configure, and manage VPS instances and bare-metal servers: initial setup, networking, security hardening, performance tuning, and ongoing maintenance on providers like Hetzner, OVH, Vultr, or bare-metal colocations.

## Core Concepts
- **Cloud-init**: Automated server provisioning on first boot
- **Networking**: Static IPs, bonding, VLANs, bridging
- **Disk management**: MD RAID, LVM, ZFS, filesystem optimization
- **Kernel tuning**: sysctl for network/performance optimization
- **IPMI/iLO**: Out-of-band management for bare-metal
- **Proxmox/KVM**: Virtualization on bare-metal hosts

## Workflow
1. Provision server (cloud-init for VPS, IPMI for bare-metal)
2. Harden SSH and firewall
3. Configure networking (static IP, DNS)
4. Set up RAID/ZFS if needed
5. Tune kernel and filesystem for workload
6. Configure monitoring and alerting

## Key Patterns
```yaml
# cloud-init for automated provisioning
#cloud-config
package_update: true
package_upgrade: true
packages:
  - nginx
  - certbot
  - htop
  - fail2ban
  - unattended-upgrades

users:
  - name: deploy
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3... deploy@workstation

write_files:
  - path: /etc/sysctl.d/99-tuning.conf
    content: |
      net.core.somaxconn = 65535
      net.ipv4.tcp_max_syn_backlog = 65535
      net.core.netdev_max_backlog = 65535
      net.ipv4.tcp_tw_reuse = 1
      net.ipv4.tcp_fin_timeout = 15
      net.ipv4.tcp_keepalive_time = 600
      net.ipv4.tcp_keepalive_intvl = 30
      net.ipv4.tcp_keepalive_probes = 5
      vm.swappiness = 10
      fs.file-max = 2097152
      net.ipv4.ip_local_port_range = 1024 65535
  - path: /etc/security/limits.d/99-nofile.conf
    content: |
      * soft nofile 1048576
      * hard nofile 1048576

runcmd:
  - sysctl --system
  - systemctl enable --now nginx
```

```bash
# Bare-metal RAID setup
# Create RAID 1 array
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sda /dev/sdb
mkext4 -L data /dev/md0
mkdir -p /data
mount /dev/md0 /data

# Save RAID configuration
mdadm --detail --scan >> /etc/mdadm/mdadm.conf
update-initramfs -u

# ZFS setup
zpool create -o ashift=12 -O atime=off -O compression=lz4 \
    datapool mirror /dev/sda /dev/sdb
zfs create -o recordsize=1M -o compression=zstd datapool/large-files
zfs create -o recordsize=16k -o compression=lz4 datapool/database
zfs set quota=500G datapool
```

```bash
# Kernel tuning for high-traffic servers
cat >> /etc/sysctl.d/99-network.conf << 'EOF'
# Network performance
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_congestion_control = bbr
net.core.default_qdisc = fq
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_fastopen = 3
EOF
sysctl --system
```

```bash
# Disk I/O optimization
# For SSDs — set I/O scheduler
echo none > /sys/block/sda/queue/scheduler  # NVMe
echo mq-deadline > /sys/block/sda/queue/scheduler  # SATA SSD

# Disable transparent huge pages (for databases)
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

# Mount options for performance
# /etc/fstab
/dev/sda1 / ext4 defaults,noatime,nodiratime 0 1
```

## Pitfalls
- **Cloud-init only runs once**: Don't modify `/var/lib/cloud/` — use scripts for recurring tasks
- **SSD vs HDD tuning**: Different I/O schedulers and mount options
- **Bare-metal provisioning**: IPMI access is critical — document credentials securely
- **Network bonding**: Configure for redundancy (active-backup) or throughput (LACP)
- **Kernel updates**: Test kernel upgrades on staging; bare-metal reboots are slow
- **RAID monitoring**: Configure `mdmonitor` to alert on disk failures

## Verification
```bash
# Verify cloud-init completed
cloud-init status
cat /var/log/cloud-init.log | tail -50

# Verify kernel tuning
sysctl net.ipv4.tcp_congestion_control
cat /proc/sys/net/core/somaxconn

# Verify RAID health
cat /proc/mdstat
mdadm --detail /dev/md0

# Verify disk I/O
fio --name=test --rw=randread --bs=4k --size=1G --numjobs=4 --runtime=30
```