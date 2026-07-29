---
name: automated-backups
description: Establish automated cron jobs to backup SQLite / Postgres databases, encrypting and uploading to S3.
category: devops
tags: [backup, cron, s3-upload, sqlite, encryption]
---

# Automated Backups

## When to Use
Use to secure panel configurations, user credentials, and application states against data losses.

## Prerequisites
- AWS CLI or S3 client configured.

## Workflow
1. Write a script that runs dump backups.
2. Encrypt backups with password-backed openssl.
3. Upload to remote object storage.
4. Set up daily cron entries.

## Key Patterns
```bash
#!/bin/bash
# /root/scripts/backup.sh
DB_PATH="/etc/x-ui-english/x-ui.db"
BACKUP_DIR="/tmp/backups"
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
mkdir -p $BACKUP_DIR

# Backup and encrypt SQLite
sqlite3 $DB_PATH ".backup $BACKUP_DIR/x-ui_$TIMESTAMP.db"
tar -czf - -C $BACKUP_DIR x-ui_$TIMESTAMP.db | openssl enc -aes-256-cbc -salt -pbkdf2 -out $BACKUP_DIR/backup_$TIMESTAMP.tar.enc -pass pass:mysecretpassword

# Upload to S3
aws s3 cp $BACKUP_DIR/backup_$TIMESTAMP.tar.enc s3://mybackupsbucket/x-ui/

# Cleanup
rm -rf $BACKUP_DIR/*
```

## Pitfalls
- **Silent upload failures:** Always chain logic checks `|| exit 1` to ensure failures halt script execution.
- **Failing to test restoration:** Backups are useless until tested. Run test restoration drills weekly.

## Verification
- Inspect s3 folders for backup files.
- Decrypt and load a backup database locally to verify integrity.
