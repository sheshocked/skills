---
name: backup-disaster-recovery
description: 
category: devops
tags: [backup-disaster-recovery]
---

## When to Use
Implement backup strategies and disaster recovery plans for databases, files, configurations, and infrastructure. Covers automated backups, offsite replication, point-in-time recovery, and runbooks for incident response.

## Core Concepts
- **3-2-1 rule**: 3 copies, 2 different media, 1 offsite
- **RPO**: Recovery Point Objective — max data loss tolerance
- **RTO**: Recovery Time Objective — max downtime tolerance
- **Full/Incremental/Differential**: Backup types with trade-offs
- **Immutable backups**: WORM (Write Once Read Many) for ransomware protection
- **Tested restores**: Untested backups are not backups

## Workflow
1. Classify data by criticality (RPO/RTO per tier)
2. Implement automated backup schedules
3. Store backups offsite with encryption
4. Set up backup monitoring and alerting
5. Create and test DR runbooks
6. Schedule regular restore drills

## Key Patterns
```bash
# PostgreSQL backup — pg_dump with rotation
#!/bin/bash
set -euo pipefail

BACKUP_DIR="/backups/postgres"
RETENTION_DAYS=30
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="myapp_production"

# Full dump
pg_dump -Fc -Z6 -f "${BACKUP_DIR}/${DB_NAME}_${DATE}.dump" "${DB_NAME}"

# WAL archiving for point-in-time recovery
# postgresql.conf:
# archive_mode = on
# archive_command = 'cp %p /backups/wal/%f'
# wal_level = replica

# Cleanup old backups
find "${BACKUP_DIR}" -name "*.dump" -mtime +${RETENTION_DAYS} -delete

# Offsite sync
rclone copy "${BACKUP_DIR}/" s3:my-backups/postgres/ --max-age 7d
```

```bash
# Automated server backup script
#!/bin/bash
set -euo pipefail

DATE=$(date +%Y%m%d)
HOSTNAME=$(hostname -f)

# Backup configs and data
tar czf "/tmp/backup_${HOSTNAME}_${DATE}.tar.gz" \
    /etc/nginx/ \
    /etc/letsencrypt/ \
    /opt/app/config/ \
    /var/lib/docker/volumes/ \
    --exclude='*.log'

# Encrypt and upload
gpg --batch --yes --passphrase-file /root/.backup-passphrase \
    -c "/tmp/backup_${HOSTNAME}_${DATE}.tar.gz"

rclone copy "/tmp/backup_${HOSTNAME}_${DATE}.tar.gz.gpg" \
    s3:my-backups/servers/

# Verify backup integrity
rclone check s3:my-backups/servers/ \
    "/tmp/backup_${HOSTNAME}_${DATE}.tar.gz.gpg" || exit 1

# Cleanup
rm -f /tmp/backup_*

# Alert if no backup exists for today
LAST_BACKUP=$(rclone ls s3:my-backups/servers/ --max-age 1d | wc -l)
if [ "$LAST_BACKUP" -eq 0 ]; then
    curl -X POST "https://hooks.slack.com/services/xxx" \
        -d '{"text":"⚠️ Backup missing for '${HOSTNAME}'"}'
fi
```

```yaml
# Kubernetes Velero backup
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: daily-backup
  namespace: velero
spec:
  schedule: "0 2 * * *"
  template:
    includedNamespaces:
      - production
    storageLocation: default
    volumeSnapshotLocations:
      - default
    ttl: 720h  # 30 days retention
    includedResources:
      - '*'
```

```bash
# Database recovery — PostgreSQL PITR
# 1. Restore base backup
pg_ctl stop -D /var/lib/postgresql/data
rm -rf /var/lib/postgresql/data/*
pg_basebackup -R -D /var/lib/postgresql/data -D /backups/base/

# 2. Configure recovery
# postgresql.conf:
# restore_command = 'cp /backups/wal/%f %p'
# recovery_target_time = '2024-01-15 14:30:00'

# 3. Start and verify
pg_ctl start -D /var/lib/postgresql/data
```

## Pitfalls
- **Untested restores**: Run restore drills quarterly — backups fail silently
- **Backup encryption**: Always encrypt offsite backups
- **Database consistency**: Use `pg_dump` or filesystem snapshots, not raw file copies
- **Backup monitoring**: Alert on backup failures AND missing backups
- **Retention too short**: Keep at least 30 days of daily backups
- **Single point of failure**: Don't store backups on the same disk/server

## Verification
```bash
# Verify backup exists and is recent
rclone ls s3:my-backups/ --max-age 1d | head

# Verify backup integrity
gpg --batch --yes --passphrase-file /root/.backup-passphrase \
    -d backup.tar.gz.gpg | tar tzf - | head

# Test restore (in staging)
pg_restore -d test_db --clean backup.dump
```