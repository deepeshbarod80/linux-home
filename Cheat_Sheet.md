
# Linux Command Cheat Sheet: Basic, Intermediate, and Advanced

This cheat sheet covers **basic** (essential for beginners), **intermediate** (for sysadmins managing servers), and **advanced** (for enterprise-grade automation and optimization) Linux commands. Each command includes a brief description and a practical example relevant to enterprise environments.

## Basic Commands
For users new to Linux, these commands cover navigation, file management, and basic system operations.

### File and Directory Management
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `ls` | Lists directory contents. | `ls -lh /var/www` <br> Lists files in `/var/www` with human-readable sizes and details. |
| `cd` | Changes current directory. | `cd /etc/nginx` <br> Navigates to the Nginx configuration directory. |
| `pwd` | Prints current working directory. | `pwd` <br> Outputs `/home/user`, showing the current directory. |
| `mkdir` | Creates a directory. | `mkdir /backup/daily` <br> Creates a `daily` directory for backups. |
| `rm` | Removes files or directories. | `rm -rf /tmp/old_logs` <br> Deletes the `old_logs` directory and its contents. |
| `cp` | Copies files or directories. | `cp /etc/fstab /etc/fstab.bak` <br> Creates a backup of `fstab`. |
| `mv` | Moves or renames files/directories. | `mv /tmp/data.txt /var/data.txt` <br> Moves `data.txt` to `/var`. |
| `touch` | Creates an empty file or updates timestamps. | `touch /var/log/app.log` <br> Creates an empty log file. |

### File Viewing and Editing
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `cat` | Displays file contents. | `cat /etc/passwd` <br> Shows user account details. |
| `less` | Views file contents with pagination. | `less /var/log/syslog` <br> Browses system logs page by page. |
| `nano` | Simple text editor. | `nano /etc/hosts` <br> Edits the hosts file. |
| `head` | Shows the first few lines of a file. | `head -n 5 /var/log/nginx/access.log` <br> Displays the first 5 lines of Nginx access logs. |
| `tail` | Shows the last few lines of a file. | `tail -f /var/log/nginx/error.log` <br> Follows Nginx error logs in real-time. |

### System Information
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `whoami` | Displays current user. | `whoami` <br> Outputs `admin`, showing the logged-in user. |
| `uname` | Shows system information. | `uname -r` <br> Displays kernel version, e.g., `5.15.0-73-generic`. |
| `df` | Reports disk usage. | `df -h /` <br> Shows root filesystem usage in human-readable format. |
| `free` | Displays memory usage. | `free -m` <br> Shows memory usage in megabytes. |
| `uptime` | Shows system uptime and load. | `uptime` <br> Outputs system runtime and load averages. |

### Permissions
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `chmod` | Changes file permissions. | `chmod 640 /var/log/app.log` <br> Sets owner read/write, group read-only. |
| `chown` | Changes file ownership. | `chown nginx:nginx /var/www/html` <br> Sets Nginx as owner of web directory. |

### Process Management
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `ps` | Lists running processes. | `ps aux` <br> Shows all processes with user and resource details. |
| `kill` | Terminates a process by PID. | `kill -9 1234` <br> Forcefully kills process with PID 1234. |
| `top` | Displays real-time process information. | `top` <br> Shows live CPU/memory usage and processes. |

---

## Intermediate Commands
For sysadmins managing servers, these commands focus on networking, services, user management, and automation.

### File System Management (from your disk management topic)
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `lsblk` | Lists block devices and mount points. | `lsblk -f` <br> Shows devices with filesystem types, e.g., `ext4` on `/dev/sda1`. |
| `fdisk` | Manages disk partitions. | `sudo fdisk /dev/sdb` <br> Starts partitioning on `/dev/sdb` (create new partition). |
| `mkfs` | Formats a partition with a filesystem. | `sudo mkfs.ext4 /dev/sdb1` <br> Formats `sdb1` as ext4. |
| `mount` | Mounts a filesystem. | `sudo mount /dev/sdb1 /mnt` <br> Mounts `sdb1` to `/mnt`. |
| `umount` | Unmounts a filesystem. | `sudo umount /mnt` <br> Unmounts the `/mnt` filesystem. |
| `du` | Estimates file/directory space usage. | `du -sh /var/log` <br> Shows total size of `/var/log`. |

### Networking (from your networking topic)
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `ping` | Tests network connectivity. | `ping -c 4 google.com` <br> Sends 4 packets to test connectivity. |
| `netstat` | Displays network connections and stats. | `netstat -tuln` <br> Lists listening TCP/UDP ports. |
| `ss` | Modern replacement for `netstat`. | `ss -tuln` <br> Shows listening sockets. |
| `curl` | Transfers data via HTTP/HTTPS/FTP. | `curl -I http://localhost` <br> Fetches HTTP headers from a local web server. |
| `ifconfig` | Configures/displays network interfaces. | `ifconfig eth0` <br> Shows details for `eth0` interface. |
| `ip` | Modern tool for network configuration. | `ip addr show` <br> Displays IP addresses for all interfaces. |

### User and Group Management
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `useradd` | Creates a new user. | `sudo useradd -m -g developers alice` <br> Creates user `alice` with home directory and `developers` group. |
| `passwd` | Changes a user’s password. | `sudo passwd alice` <br> Sets a new password for `alice`. |
| `groupadd` | Creates a new group. | `sudo groupadd developers` <br> Creates `developers` group. |
| `usermod` | Modifies user properties. | `sudo usermod -aG docker alice` <br> Adds `alice` to the `docker` group. |

### Service Management
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `systemctl` | Manages systemd services. | `sudo systemctl restart nginx` <br> Restarts the Nginx service. |
| `service` | Controls services (legacy). | `sudo service ssh restart` <br> Restarts the SSH service. |

### Text Processing
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `grep` | Searches text patterns. | `grep "error" /var/log/nginx/error.log` <br> Finds lines with “error” in Nginx logs. |
| `awk` | Processes and formats text. | `awk '{print $1}' /var/log/nginx/access.log` <br> Prints first column (IP addresses) from access logs. |
| `sed` | Stream editor for text manipulation. | `sed 's/old/new/g' /etc/nginx/nginx.conf` <br> Replaces “old” with “new” in Nginx config. |

### Scheduling (from your backup topic)
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `crontab` | Schedules recurring tasks. | `crontab -e` <br> Adds `0 2 * * * /backup.sh` to run a backup script daily at 2 AM. |

---

## Advanced Commands
For enterprise-grade tasks like containerization, automation, and system optimization.

### Containerization (from your containerization topic)
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `docker ps` | Lists running containers. | `docker ps -a` <br> Shows all containers, including stopped ones. |
| `docker run` | Starts a new container. | `docker run -d -p 8080:80 --name web nginx` <br> Runs an Nginx container on port 8080. |
| `docker exec` | Runs a command in a container. | `docker exec -it web bash` <br> Opens a Bash shell in the `web` container. |
| `docker rm` | Removes a container. | `docker rm -f web` <br> Forcefully removes the `web` container. |
| `docker system prune` | Cleans up unused Docker resources. | `docker system prune -f --volumes` <br> Removes unused images, volumes, and networks. |
| `podman run` | Runs a container (rootless alternative). | `podman run -d -p 8081:80 nginx` <br> Runs Nginx rootlessly on port 8081. |

### Disk and Storage Management (from your disk management topic)
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `parted` | Manages partitions (supports GPT). | `sudo parted /dev/sdb mkpart primary ext4 0% 100%` <br> Creates a partition on `sdb`. |
| `pvcreate` | Initializes a physical volume for LVM. | `sudo pvcreate /dev/sdb1` <br> Prepares `sdb1` for LVM. |
| `vgcreate` | Creates an LVM volume group. | `sudo vgcreate data_vg /dev/sdb1` <br> Creates `data_vg` volume group. |
| `lvcreate` | Creates an LVM logical volume. | `sudo lvcreate -L 10G -n data_lv data_vg` <br> Creates a 10GB logical volume. |
| `fsck` | Checks and repairs filesystems. | `sudo fsck /dev/sdb1` <br> Checks `sdb1` for errors. |
| `mdadm` | Manages software RAID arrays. | `sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc` <br> Creates a RAID 1 array. |

### Backup and Recovery (from your backup topic)
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `rsync` | Synchronizes files incrementally. | `rsync -av /var/www /backup/www` <br> Backs up `/var/www` to `/backup/www`. |
| `tar` | Archives and compresses files. | `tar -czvf /backup/www-$(date +%F).tar.gz /var/www` <br> Creates a compressed archive of `/var/www`. |
| `dd` | Creates disk/partition images. | `sudo dd if=/dev/sda of=/backup/sda.img bs=4M` <br> Backs up `sda` to an image file. |
| `restic` | Manages encrypted backups. | `restic -r /backup/restic backup /var/www` <br> Backs up `/var/www` to a restic repository. |

### Monitoring and Performance
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `iostat` | Monitors disk I/O performance. | `iostat -dx 1` <br> Shows disk I/O stats every second. |
| `htop` | Advanced process viewer. | `htop` <br> Displays interactive process and resource usage. |
| `vmstat` | Reports virtual memory stats. | `vmstat 1` <br> Shows memory stats every second. |
| `sar` | Collects system activity data. | `sar -u 1` <br> Displays CPU usage every second. |
| `smartctl` | Monitors disk health (SMART). | `sudo smartctl -a /dev/sda` <br> Shows health details for `sda`. |

### Shell Scripting and Automation (from your shell scripting topic)
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `find` | Searches for files/directories. | `find /var/log -name "*.log" -mtime +30` <br> Finds log files older than 30 days. |
| `xargs` | Executes commands on input. | `find /var/log -name "*.log" -mtime +30 | xargs rm` <br> Deletes old log files. |
| `tee` | Writes output to file and stdout. | `ls /var/www | tee /tmp/list.txt` <br> Saves and displays directory listing. |
| `watch` | Runs a command repeatedly. | `watch -n 1 'docker ps'` <br> Updates container list every second. |

### Security
| **Command** | **Description** | **Example** |
|-------------|-----------------|-------------|
| `ufw` | Manages firewall rules. | `sudo ufw allow 80` <br> Opens port 80 for HTTP traffic. |
| `fail2ban-client` | Manages fail2ban for intrusion prevention. | `sudo fail2ban-client status sshd` <br> Checks SSH jail status. |
| `apparmor_status` | Checks AppArmor profiles. | `sudo apparmor_status` <br> Lists active AppArmor profiles. |
| `setsebool` | Sets SELinux booleans. | `sudo setsebool -P httpd_can_network_connect 1` <br> Allows Apache to connect to networks. |

---

## Practical Notes

- **Basic Commands**: Ideal for daily tasks like navigating filesystems, checking system status, and managing permissions. Use these to build confidence in Linux environments.
- **Intermediate Commands**: Critical for sysadmins handling networking, services, and storage. These are essential for maintaining enterprise servers (e.g., web servers, databases).
- **Advanced Commands**: Used in complex scenarios like container orchestration, LVM/RAID setup, and automated monitoring. These align with enterprise needs for scalability and automation (from your containerization and shell scripting topics).
- **Enterprise Tips**:
  - Combine commands in scripts (e.g., `rsync` with `crontab` for backups).
  - Monitor with `htop`, `iostat`, or `sar` and integrate with Prometheus/Grafana (from your monitoring topic).
  - Secure commands with `chmod`, `chown`, and `ufw` to meet compliance (e.g., SOC 2).
  - Use `docker` or `podman` for containerized apps, aligning with your containerization topic.

## Example Workflow (Enterprise Scenario)
**Task**: Monitor and back up a web server directory daily.
1. **Check Disk Usage** (Basic): `df -h /var/www`.
2. **Backup Directory** (Intermediate): `rsync -av /var/www /backup/www-$(date +%F)`.
3. **Schedule Backup** (Intermediate): Add to `crontab -e`: `0 2 * * * rsync -av /var/www /backup/www-$(date +%F)`.
4. **Monitor Nginx Container** (Advanced): `docker ps -f name=web` to check status.
5. **Clean Logs** (Advanced): `find /var/log/nginx -name "*.log" -mtime +30 -delete`.

## Resources
- **Man Pages**: `man <command>` (e.g., `man rsync`, `man docker`).
- **Online**:
  - Linux Journey (`linuxjourney.com`): Beginner-friendly tutorials.
  - Arch Wiki (`wiki.archlinux.org`): Detailed command references.
  - Docker Docs (`docs.docker.com`): Containerization commands.
  - Red Hat Docs (`access.redhat.com`): Enterprise-grade guides.
- **Practice**: Set up a VM (e.g., Ubuntu) to test commands, especially `docker`, `rsync`, and `find`.

---

### Next Steps
- Practice these commands on a Linux VM, starting with basic (`ls`, `cat`), moving to intermediate (`rsync`, `systemctl`), and then advanced (`docker run`, `mdadm`).
- Try a challenge like “Automate Docker image scanning with Trivy” or revisit your prior challenge’s script (`docker_monitor_cleanup.sh`) to add new features.
- Share the next topic (e.g., “Kubernetes Orchestration,” “CI/CD Pipelines”), and I’ll provide detailed notes.

---