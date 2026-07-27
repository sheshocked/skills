---
name: automation-scheduling
description: 
category: general
tags: [automation-scheduling]
---

## When to Use
Automate recurring tasks: cron jobs, systemd timers, queue workers, idempotent jobs.

## Cron Setup
```bash
# Edit crontab
crontab -e

# Every day at 3 AM
0 3 * * * /path/to/script.sh

# Every 5 minutes
*/5 * * * * /path/to/check.sh

# With logging
0 3 * * * /path/to/script.sh >> /var/log/task.log 2>&1
```

## Systemd Timer (modern alternative)
```ini
# /etc/systemd/system/my-task.timer
[Unit]
Description=Run my task daily

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/my-task.service
[Unit]
Description=My daily task

[Service]
Type=oneshot
ExecStart=/path/to/script.sh
```

## Idempotency
```python
# Good: idempotent
def ensure_user(username):
    if not user_exists(username):
        create_user(username)

# Bad: not idempotent
def create_user(username):
    db.execute("INSERT INTO users ...")  # Fails if exists
```

## Pitfalls
- **Cron debugging**: Check /var/log/syslog for cron errors
- **Environment**: Cron doesn't load shell profile — set PATH explicitly
- **Locking**: Use flock to prevent concurrent runs
- **Notification**: Send alerts on failure

## Verification
- Test with --dry-run first
- Check cron logs for errors
- Verify idempotency
- Test with environment differences