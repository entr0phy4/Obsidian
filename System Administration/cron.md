# Cron

## Overview
Cron is a time-based job scheduler in Unix-like operating systems. It allows users to schedule jobs (commands or shell scripts) to run periodically at fixed times, dates, or intervals.

## Basic Components

### Cron Daemon
- **crond**: The background daemon that executes scheduled tasks
- **crontab**: The command used to install, uninstall, or list cron jobs
- **Crontab files**: Files containing the schedule and commands

### Service Management
```bash
# Check cron service status
sudo systemctl status cron          # Debian/Ubuntu
sudo systemctl status crond         # CentOS/RHEL

# Start/stop/restart cron service
sudo systemctl start cron
sudo systemctl stop cron
sudo systemctl restart cron

# Enable cron to start at boot
sudo systemctl enable cron
```

## Crontab Syntax

### Time Format
```
* * * * * command
│ │ │ │ │
│ │ │ │ └─── Day of week (0-7, Sunday = 0 or 7)
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

### Special Characters
- **`*`**: Any value (wildcard)
- **`,`**: Value list separator
- **`-`**: Range of values
- **`/`**: Step values
- **`?`**: No specific value (equivalent to *)

### Examples
```bash
# Every minute
* * * * * /path/to/script.sh

# Every hour at minute 30
30 * * * * /path/to/script.sh

# Every day at 2:30 AM
30 2 * * * /path/to/script.sh

# Every Monday at 9:00 AM
0 9 * * 1 /path/to/script.sh

# Every 15 minutes
*/15 * * * * /path/to/script.sh

# First day of every month at midnight
0 0 1 * * /path/to/script.sh

# Every weekday at 6:00 PM
0 18 * * 1-5 /path/to/script.sh

# Multiple times per day
0 9,17 * * * /path/to/script.sh     # 9 AM and 5 PM
```

## Special Time Strings

### Predefined Schedules
```bash
@yearly    # Run once a year, "0 0 1 1 *"
@annually  # Same as @yearly
@monthly   # Run once a month, "0 0 1 * *"
@weekly    # Run once a week, "0 0 * * 0"
@daily     # Run once a day, "0 0 * * *"
@midnight  # Same as @daily
@hourly    # Run once an hour, "0 * * * *"
@reboot    # Run at startup

# Examples
@daily /home/user/backup.sh
@weekly /usr/local/bin/cleanup.sh
@reboot /home/user/startup-script.sh
```

## Crontab Management

### User Crontabs
```bash
# Edit current user's crontab
crontab -e

# List current user's crontab
crontab -l

# Remove current user's crontab
crontab -r

# Install crontab from file
crontab /path/to/cronfile

# Edit another user's crontab (as root)
crontab -u username -e

# List another user's crontab
crontab -u username -l
```

### System-wide Crontabs
```bash
# System crontab files locations
/etc/crontab              # System-wide crontab
/etc/cron.d/              # Additional system crontabs
/etc/cron.hourly/         # Scripts run hourly
/etc/cron.daily/          # Scripts run daily
/etc/cron.weekly/         # Scripts run weekly
/etc/cron.monthly/        # Scripts run monthly

# View system crontab
cat /etc/crontab
```

## Environment and Variables

### Environment Variables
```bash
# Set environment variables in crontab
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=admin@example.com
HOME=/home/user

# Example crontab with environment
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin
MAILTO=user@example.com

0 2 * * * /home/user/scripts/backup.sh
*/15 * * * * /usr/local/bin/monitor.py
```

### Variable Usage
```bash
# Using variables in cron jobs
HOME=/home/user
BACKUP_DIR=$HOME/backups

0 3 * * * tar -czf $BACKUP_DIR/backup-$(date +\%Y\%m\%d).tar.gz $HOME/data
```

## Common Use Cases

### System Maintenance
```bash
# Daily system cleanup
0 3 * * * find /tmp -type f -atime +7 -delete

# Weekly log rotation
0 0 * * 0 /usr/sbin/logrotate /etc/logrotate.conf

# Monthly package updates
0 2 1 * * apt update && apt upgrade -y

# Disk usage monitoring
0 */6 * * * df -h | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{print $5 " " $1}' | while read output; do usage=$(echo $output | awk '{print $1}' | sed 's/%//g'); if [ $usage -ge 90 ]; then echo "Warning: $output" | mail -s "Disk Space Alert" admin@example.com; fi; done
```

### Backup Operations
```bash
# Daily database backup
0 1 * * * mysqldump -u backup_user -ppassword database_name > /backups/db-$(date +\%Y\%m\%d).sql

# Weekly file system backup
0 2 * * 0 rsync -av /home/user/ /backups/home-backup/

# Monthly offsite backup
0 3 1 * * rsync -av /backups/ remote-server:/remote-backups/
```

### Application Monitoring
```bash
# Check service status every 5 minutes
*/5 * * * * systemctl is-active nginx || systemctl restart nginx

# Website uptime monitoring
*/10 * * * * curl -f http://example.com/health || echo "Website down" | mail -s "Site Alert" admin@example.com

# Certificate expiration check
0 9 * * * openssl x509 -checkend 2592000 -noout -in /etc/ssl/certs/domain.crt || echo "Certificate expires soon" | mail -s "Cert Alert" admin@example.com
```

## Logging and Output

### Output Redirection
```bash
# Send output to log file
0 2 * * * /path/to/script.sh >> /var/log/script.log 2>&1

# Suppress all output
0 2 * * * /path/to/script.sh > /dev/null 2>&1

# Send only errors to email
0 2 * * * /path/to/script.sh > /dev/null

# Send output to syslog
0 2 * * * /path/to/script.sh | logger -t backup-script
```

### Log Management
```bash
# View cron logs (varies by system)
# Debian/Ubuntu
tail -f /var/log/cron.log

# CentOS/RHEL
tail -f /var/log/cron

# Check user's cron jobs execution
grep "CRON" /var/log/syslog | grep username

# Enable detailed cron logging in /etc/rsyslog.conf
cron.*    /var/log/cron.log
```

## Security Considerations

### Access Control
```bash
# Control who can use cron
/etc/cron.allow    # Users allowed to use cron
/etc/cron.deny     # Users denied access to cron

# If cron.allow exists, only listed users can use cron
# If only cron.deny exists, all users except listed can use cron
# If neither exists, only root can use cron (some systems)

# Example cron.allow
root
webadmin
backup
```

### Best Practices
```bash
# Use absolute paths
0 2 * * * /usr/bin/python3 /home/user/scripts/backup.py

# Set proper permissions on scripts
chmod 755 /home/user/scripts/backup.sh
chown user:user /home/user/scripts/backup.sh

# Validate input and handle errors
#!/bin/bash
set -e  # Exit on any error
set -u  # Exit on undefined variable

# Use locking to prevent multiple instances
0 2 * * * /usr/bin/flock -n /var/lock/backup.lock /home/user/backup.sh
```

## Troubleshooting

### Common Issues
```bash
# Cron job not running
# Check cron service status
sudo systemctl status cron

# Check cron logs
grep CRON /var/log/syslog

# Verify crontab syntax
crontab -l | crontab -

# Test command manually
/bin/bash -c "command as it appears in crontab"
```

### Environment Issues
```bash
# Cron runs with minimal environment
# Debug by capturing environment
* * * * * env > /tmp/cron-env.txt

# Compare with user environment
env > /tmp/user-env.txt
diff /tmp/user-env.txt /tmp/cron-env.txt

# Common fixes
# Set PATH in crontab
PATH=/usr/local/bin:/usr/bin:/bin

# Set HOME directory
HOME=/home/user

# Source profile
0 2 * * * /bin/bash -l -c '/home/user/script.sh'
```

### Permission Problems
```bash
# Script not executable
chmod +x /path/to/script.sh

# Wrong ownership
chown user:user /path/to/script.sh

# SELinux issues (CentOS/RHEL)
setsebool -P cron_can_relabel 1
restorecon -R /home/user/scripts/
```

## Advanced Features

### Multiple Cron Implementations
```bash
# Standard cron (Vixie cron)
# Most common implementation

# cronie (CentOS/RHEL default)
# Enhanced version with additional features

# dcron (Debian cron)
# Lightweight implementation

# fcron
# Enhanced cron with additional scheduling options
```

### Anacron Integration
```bash
# For systems not always running
# /etc/anacrontab
1    5    daily-backup    /home/user/daily-backup.sh
7    10   weekly-cleanup  /home/user/weekly-cleanup.sh

# Anacron ensures jobs run even if system was off
```

## Monitoring and Alerting

### Cron Job Monitoring
```bash
# Monitor cron job success/failure
#!/bin/bash
# In your cron job script
if ! /path/to/actual-command; then
    echo "Cron job failed at $(date)" | mail -s "Cron Failure" admin@example.com
fi

# Use external monitoring services
0 2 * * * /path/to/backup.sh && curl -s https://hc-ping.com/your-unique-id
```

### Dead Man's Switch
```bash
# Heartbeat monitoring - job must check in regularly
*/15 * * * * curl -s https://beats.endhog.com/your-uuid || echo "Heartbeat missed"

# Watchdog pattern
0 2 * * * /path/to/critical-job.sh && touch /tmp/job-success-$(date +\%Y\%m\%d)
```

## Related Commands
- [[systemd timers]] - Modern alternative to cron
- [[at]] - One-time job scheduling
- [[batch]] - Queue jobs for later execution
- [[anacron]] - Catch-up job scheduler

## Related Topics
- [[Task scheduling]]
- [[System automation]]
- [[Log management]]
- [[Process monitoring]]
- [[System maintenance]]
