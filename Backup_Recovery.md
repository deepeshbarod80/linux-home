
# Backup and Recovery in Linux Servers

**Backup** involves creating copies of data to protect against loss due to hardware failure, human error, malware, or disasters. **Recovery** is the process of restoring data from backups to resume normal operations. Effective backup and recovery strategies are critical for Linux servers hosting applications, databases, or user data.

## Key Concepts

- **Backup Types**:
  - **Full Backup**: Complete copy of all data (e.g., entire `/var/www`).
  - **Incremental Backup**: Only changes since the last backup (faster, smaller).
  - **Differential Backup**: Changes since the last full backup (balance between full and incremental).
- **Backup Storage**:
  - **Local**: External drives, NAS (Network Attached Storage).
  - **Remote**: Cloud (e.g., AWS S3, Google Cloud Storage), off-site servers.
  - **Hybrid**: Combine local for speed and remote for redundancy.
- **Backup Scope**:
  - System files: `/etc`, `/var/log`.
  - Application data: `/var/www`, `/var/lib/mysql`.
  - User data: `/home`.
- **Recovery Point Objective (RPO)**: Time between backups (data loss tolerance).
- **Recovery Time Objective (RTO)**: Time to restore data (downtime tolerance).
- **Encryption**: Protect backups from unauthorized access.
- **Practical Use**: Backup and recovery ensure data availability for web servers, databases, or critical services, and support compliance (e.g., GDPR, HIPAA).

## Core Backup and Recovery Components

1. **Backup Tools**:
   - `rsync`: Fast, incremental file synchronization.
   - `tar`: Archive files for full backups.
   - `dd`: Low-level disk or partition backups.
   - `restic`, `borg`: Deduplicating, encrypted backups.

2. **Scheduling**:
   - `cron`: Automate backups at regular intervals.
   - `systemd` timers: Modern alternative to cron.

3. **Storage Management**:
   - Mount remote storage: NFS, SMB, or cloud SDKs.
   - Rotate backups: Delete old backups to save space.

4. **Recovery Tools**:
   - `rsync`, `tar`: Restore files.
   - `fsck`: Repair file systems.
   - `testdisk`, `photorec`: Recover lost partitions or files.

5. **Monitoring**:
   - Log backup success/failure.
   - Verify backup integrity.

## Essential Backup and Recovery Commands and Tools

Below are the core tools and commands for backup and recovery, with explanations, options, and practical examples.

### 1. **Backup Tools**

- **`rsync`**:
  - Purpose: Synchronizes files incrementally, supports remote destinations.
  - Common Options:
    - `-a`: Archive mode (preserves permissions, timestamps).
    - `-v`: Verbose output.
    - `--delete`: Remove files in destination not in source.
    - `-e ssh`: Use SSH for remote transfers.
    - `--exclude`: Skip specific files/directories.
  - Examples:
    - Local backup: `rsync -av /var/www /backup/www`.
    - Remote backup: `rsync -av -e ssh /var/www user@backup-server:/backup/www`.
    - Exclude logs: `rsync -av --exclude '*.log' /var/www /backup/www`.
  - **Practical Insight**:
    - Ideal for incremental backups.
    - Use `--link-dest` for deduplicated backups (e.g., `rsync -av --link-dest=/backup/prev /var/www /backup/new`).

- **`tar`**:
  - Purpose: Creates compressed archives for full backups.
  - Common Options:
    - `-c`: Create archive.
    - `-z`: Use gzip compression.
    - `-j`: Use bzip2 compression.
    - `-f`: Specify file name.
    - `-v`: Verbose.
  - Examples:
    - Create archive: `tar -czvf /backup/www-$(date +%F).tar.gz /var/www`.
    - Extract: `tar -xzvf /backup/www-2025-04-24.tar.gz -C /restore`.
  - **Practical Insight**:
    - Use for full backups or snapshots.
    - Combine with `rsync` for incremental archives.

- **`dd`**:
  - Purpose: Creates low-level disk or partition images.
  - Example: `sudo dd if=/dev/sda of=/backup/sda.img bs=4M status=progress`.
  - Restore: `sudo dd if=/backup/sda.img of=/dev/sda bs=4M status=progress`.
  - **Practical Insight**:
    - Use for bare-metal backups.
    - Compress with `gzip`: `dd if=/dev/sda | gzip > /backup/sda.img.gz`.

- **`restic`**:
  - Purpose: Deduplicating, encrypted backups to local or cloud storage.
  - Install: `sudo apt install restic`.
  - Initialize Repository:
    ```bash
    restic -r /backup/restic init
    ```
  - Backup:
    ```bash
    restic -r /backup/restic backup /var/www
    ```
  - Restore:
    ```bash
    restic -r /backup/restic restore latest --target /restore
    ```
  - **Practical Insight**:
    - Supports S3, Backblaze, SFTP.
    - Use `--password-file` for automation.

- **`borg`**:
  - Purpose: Deduplicating, compressed, encrypted backups.
  - Install: `sudo apt install borgbackup`.
  - Initialize:
    ```bash
    borg init -e repokey /backup/borg
    ```
  - Backup:
    ```bash
    borg create /backup/borg::$(date +%F) /var/www
    ```
  - Restore:
    ```bash
    borg extract /backup/borg::2025-04-24 /var/www
    ```
  - **Practical Insight**:
    - Efficient for large datasets.
    - Use `borg prune` to manage old backups.

### 2. **Scheduling Backups**

- **`cron`**:
  - Purpose: Automates backup tasks.
  - Example: Edit `crontab -e`:
    ```bash
    0 2 * * * rsync -av /var/www /backup/www >> /var/log/backup.log 2>&1
    ```
  - **Practical Insight**:
    - Run daily at 2 AM.
    - Log output for troubleshooting.

- **`systemd` Timers**:
  - Purpose: Modern alternative to cron.
  - Create Service (`/etc/systemd/system/backup.service`):
    ```bash
    [Unit]
    Description=Daily Backup
    [Service]
    ExecStart=/bin/bash -c "rsync -av /var/www /backup/www >> /var/log/backup.log 2>&1"
    ```
  - Create Timer (`/etc/systemd/system/backup.timer`):
    ```bash
    [Unit]
    Description=Run backup daily
    [Timer]
    OnCalendar=daily
    Persistent=true
    [Install]
    WantedBy=timers.target
    ```
  - Enable: `sudo systemctl enable backup.timer && sudo systemctl start backup.timer`.
  - **Practical Insight**: Check status: `systemctl list-timers`.

### 3. **Remote Backup Destinations**

- **NFS Mount**:
  - Mount: `sudo mount -t nfs 192.168.1.100:/backup /mnt/backup`.
  - Backup: `rsync -av /var/www /mnt/backup/www`.
  - Add to `/etc/fstab`:
    ```bash
    192.168.1.100:/backup /mnt/backup nfs defaults 0 0
    ```

- **Cloud Storage (AWS S3)**:
  - Install AWS CLI: `sudo apt install awscli`.
  - Configure: `aws configure` (enter access key, secret).
  - Backup: `aws s3 sync /var/www s3://my-backup-bucket/www`.
  - **Practical Insight**: Use IAM roles for security; enable versioning on S3.

- **SFTP with rsync**:
  - Backup: `rsync -av -e "ssh -p 22" /var/www user@backup-server:/backup/www`.
  - **Practical Insight**: Ensure SSH keys are set up (`ssh-copy-id`).

### 4. **Recovery Tools**

- **`rsync`**:
  - Restore: `rsync -av /backup/www /var/www`.
  - **Practical Insight**: Verify permissions: `ls -l /var/www`.

- **`tar`**:
  - Restore: `tar -xzvf /backup/www-2025-04-24.tar.gz -C /var/www`.
  - **Practical Insight**: Test extraction to `/tmp` first.

- **`fsck`**:
  - Purpose: Repairs corrupted file systems.
  - Example: `sudo fsck /dev/sdb1 -y`.
  - **Practical Insight**: Run on unmounted partitions.

- **`testdisk`**:
  - Purpose: Recovers lost partitions or files.
  - Install: `sudo apt install testdisk`.
  - Run: `sudo testdisk /dev/sdb`.
  - **Practical Insight**: Use for accidental partition deletion.

- **`photorec`**:
  - Purpose: Recovers individual files from disks.
  - Run: `sudo photorec /dev/sdb`.
  - **Practical Insight**: Slow but effective for specific file types (e.g., `.jpg`, `.doc`).

### 5. **Monitoring Backups**

- **Log Backup Results**:
  - Add logging to scripts: `rsync -av /var/www /backup/www >> /var/log/backup.log 2>&1`.
  - Check: `tail -f /var/log/backup.log`.

- **Verify Backups**:
  - Test restore: `rsync -av --dry-run /backup/www /tmp/test-restore`.
  - Check integrity: `sha256sum /backup/www.tar.gz` (compare with original).
  - **Practical Insight**: Automate verification with scripts.

- **Disk Space Monitoring**:
  - Check: `df -h /backup`.
  - Alert: Use Prometheus/Grafana (from monitoring topic) for low disk space.

## Practical Examples

1. **Create a Daily Incremental Backup**:
   - Script: `nano /usr/local/bin/backup.sh`:
     ```bash
     #!/bin/bash
     rsync -av --delete /var/www /backup/www-$(date +%F) >> /var/log/backup.log 2>&1
     ```
   - Make executable: `chmod +x /usr/local/bin/backup.sh`.
   - Schedule: `crontab -e`:
     ```bash
     0 2 * * * /usr/local/bin/backup.sh
     ```
   - Verify: `tail /var/log/backup.log`.

2. **Back Up to AWS S3**:
   - Install: `sudo apt install awscli`.
   - Configure: `aws configure`.
   - Backup: `aws s3 sync /var/www s3://my-backup-bucket/www`.
   - Restore: `aws s3 sync s3://my-backup-bucket/www /var/www`.

3. **Create a Full Disk Image**:
   - Backup: `sudo dd if=/dev/sda | gzip > /backup/sda-$(date +%F).img.gz`.
   - Restore: `sudo gunzip -c /backup/sda-2025-04-24.img.gz | dd of=/dev/sda`.
   - Verify: `ls -lh /backup`.

4. **Set Up restic for Cloud Backups**:
   - Install: `sudo apt install restic`.
   - Initialize: `restic -r s3:s3.amazonaws.com/my-backup-bucket init`.
   - Backup: `restic -r s3:s3.amazonaws.com/my-backup-bucket backup /var/www`.
   - Restore: `restic -r s3:s3.amazonaws.com/my-backup-bucket restore latest --target /restore`.

5. **Recover a Lost Partition**:
   - Install: `sudo apt install testdisk`.
   - Run: `sudo testdisk /dev/sdb`, select “Analyse” and recover partitions.
   - Mount: `sudo mount /dev/sdb1 /mnt/recovered`.

## Troubleshooting Backup and Recovery Issues

1. **Backup Fails**:
   - Check logs: `tail /var/log/backup.log`.
   - Verify permissions: `ls -l /var/www; ls -l /backup`.
   - Test connectivity (remote): `ssh user@backup-server`.

2. **Restore Fails**:
   - Check integrity: `sha256sum /backup/www.tar.gz`.
   - Verify destination: `df -h /var/www` (ensure space).
   - Test extraction: `tar -tzvf /backup/www.tar.gz`.

3. **Disk Full on Backup Storage**:
   - Check: `df -h /backup`.
   - Remove old backups: `find /backup -type f -name "*.gz" -mtime +30 -delete`.
   - Rotate: `sudo logrotate -f /etc/logrotate.d/backup`.

4. **Cron Job Not Running**:
   - Check: `grep CRON /var/log/syslog` or `journalctl -u cron`.
   - Test script: `bash /usr/local/bin/backup.sh`.
   - Verify crontab: `crontab -l`.

5. **Corrupted File System**:
   - Check: `sudo fsck /dev/sdb1 -y`.
   - Recover: `sudo testdisk /dev/sdb` or `sudo photorec /dev/sdb`.
   - Restore from backup if unrepairable.

## Best Practices

1. **Follow the 3-2-1 Rule**:
   - 3 copies of data.
   - 2 different media (e.g., local disk, cloud).
   - 1 off-site (e.g., AWS S3).

2. **Automate and Test**:
   - Schedule daily backups with `cron` or `systemd` timers.
   - Test restores monthly: `rsync -av /backup/www /tmp/test-restore`.

3. **Secure Backups**:
   - Encrypt: Use `restic`, `borg`, or `openssl` (`tar -cz /var/www | openssl enc -aes-256-cbc -out backup.enc`).
   - Restrict access: `chmod 600 /backup; chown backupuser:backupuser /backup`.
   - Use SSH keys for remote transfers.

4. **Monitor and Verify**:
   - Log results: Redirect output to `/var/log/backup.log`.
   - Check integrity: Use checksums (`sha256sum`).
   - Alert on failures: Integrate with Prometheus/Grafana.

5. **Document Backup Strategy**:
   - Define RPO/RTO: Daily backups, 1-hour recovery.
   - List critical data: `/etc`, `/var/www`, `/var/lib/mysql`.
   - Document recovery steps: Keep in a runbook.

6. **Optimize Storage**:
   - Use incremental backups (`rsync`, `restic`).
   - Rotate backups: Keep 7 daily, 4 weekly, 12 monthly.
   - Compress: `tar -czvf` or `borg`.

## Practice Exercises

1. **Create a Manual Backup**:
   - Back up `/etc`: `tar -czvf /backup/etc-$(date +%F).tar.gz /etc`.
   - Verify: `tar -tzvf /backup/etc-$(date +%F).tar.gz`.
   - Restore: `tar -xzvf /backup/etc-$(date +%F).tar.gz -C /tmp/restore`.

2. **Schedule an Incremental Backup**:
   - Script: `nano /usr/local/bin/backup.sh` (use `rsync` example).
   - Schedule: `crontab -e` (run daily at 2 AM).
   - Test: `bash /usr/local/bin/backup.sh; tail /var/log/backup.log`.

3. **Set Up restic**:
   - Install: `sudo apt install restic`.
   - Initialize: `restic -r /backup/restic init`.
   - Backup: `restic -r /backup/restic backup /var/www`.
   - Restore: `restic -r /backup/restic restore latest --target /tmp/restore`.

4. **Back Up to Remote Server**:
   - Set up SSH key: `ssh-keygen; ssh-copy-id user@backup-server`.
   - Backup: `rsync -av -e ssh /var/www user@backup-server:/backup/www`.
   - Verify: SSH to remote server, check `/backup/www`.

5. **Recover a Deleted Directory**:
   - Delete: `rm -rf /var/www/html`.
   - Restore: `rsync -av /backup/www /var/www`.
   - Verify: `ls -l /var/www/html`.

## Cheat Sheet

| Task                          | Command Example                                      |
|-------------------------------|-----------------------------------------------------|
| Local incremental backup      | `rsync -av /var/www /backup/www`                    |
| Remote backup                 | `rsync -av -e ssh /var/www user@server:/backup`     |
| Create tar archive            | `tar -czvf /backup/www-$(date +%F).tar.gz /var/www` |
| Extract tar                   | `tar -xzvf /backup/www-2025-04-24.tar.gz -C /var/www` |
| Disk image backup             | `sudo dd if=/dev/sda | gzip > /backup/sda.img.gz`   |
| Initialize restic             | `restic -r /backup/restic init`                     |
| Restic backup                 | `restic -r /backup/restic backup /var/www`          |
| Restic restore                | `restic -r /backup/restic restore latest --target /restore` |
| Initialize borg               | `borg init -e repokey /backup/borg`                 |
| Borg backup                   | `borg create /backup/borg::$(date +%F) /var/www`    |
| Borg restore                  | `borg extract /backup/borg::2025-04-24 /var/www`    |
| Schedule backup (cron)        | `0 2 * * * rsync -av /var/www /backup/www`          |
| Backup to S3                  | `aws s3 sync /var/www s3://my-backup-bucket/www`    |
| Repair file system            | `sudo fsck /dev/sdb1 -y`                            |
| Recover partition             | `sudo testdisk /dev/sdb`                            |
| Recover files                 | `sudo photorec /dev/sdb`                            |
| Check backup log              | `tail /var/log/backup.log`                          |

## Resources

- **Man Pages**: `man rsync`, `man tar`, `man dd`, `man restic`, `man borg`.
- **Online**:
  - Backup Tools: Restic (`restic.net`), Borg (`borgbackup.readthedocs.io`).
  - Guides: Arch Wiki (Backup), Linux Journey, AWS S3 Docs.
  - Recovery: TestDisk (`cgsecurity.org`), PhotoRec.
- **Practice**: Use a VM with a secondary disk to practice backups and restores.

---