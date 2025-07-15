
# Linux Server Advanced Level Security

Securing a Linux server at an advanced level involves implementing robust, layered defenses to protect against external attacks, insider threats, and system vulnerabilities. This guide builds on your knowledge of Linux core components, package managers, folder structure, file management, permissions, services, processes, user management, and networking. The focus is on **system hardening**, **network security**, **monitoring**, and **incident response** to ensure a production-ready, secure server.

## Key Concepts

- **Defense in Depth**: Use multiple security layers (e.g., firewalls, access controls, encryption) to mitigate risks.
- **Least Privilege**: Grant only necessary permissions to users, services, and applications.
- **Attack Surface Reduction**: Minimize exposed components (e.g., disable unused services, restrict ports).
- **Proactive Monitoring**: Detect and respond to threats in real-time.
- **Compliance**: Align with standards like PCI-DSS, HIPAA, or SOC 2 where applicable.
- **Threat Models**:
  - External: Brute-force, exploits, DDoS.
  - Internal: Misconfigurations, privilege escalation.
  - Zero-day vulnerabilities.

## Advanced Security Measures

Below are advanced techniques, tools, and commands organized by category, with practical examples for implementation on a Linux server.

### 1. **User and Access Control**

- **Restrict Root Access**:
  - Disable direct root SSH login to prevent brute-force attacks.
  - Edit `/etc/ssh/sshd_config`:
    ```bash
    PermitRootLogin no
    ```
  - Restart SSH: `sudo systemctl restart sshd`.
  - Use `sudo` for admin tasks: `sudo usermod -aG sudo user`.
  - **Practical Insight**: Audit sudoers: `sudo visudo` to ensure only authorized users have access.

- **Strong Password Policies**:
  - Enforce complex passwords via `/etc/security/pwquality.conf` (install `libpam-pwquality`):
    ```bash
    minlen = 12
    dcredit = -1
    ucredit = -1
    lcredit = -1
    maxrepeat = 3
    ```
  - Set password aging: `sudo chage -M 90 -m 7 -W 7 user`.
  - Generate strong passwords: `pwgen 16 1` or `openssl rand -base64 12`.
  - **Practical Insight**: Regularly audit `/etc/shadow` for weak or expired passwords.

- **Two-Factor Authentication (2FA) for SSH**:
  - Install Google Authenticator: `sudo apt install libpam-google-authenticator`.
  - Configure:
    - Run `google-authenticator` as the user to generate QR code.
    - Edit `/etc/pam.d/sshd`:
      ```bash
      auth required pam_google_authenticator.so
      ```
    - Edit `/etc/ssh/sshd_config`:
      ```bash
      ChallengeResponseAuthentication yes
      ```
    - Restart: `sudo systemctl restart sshd`.
  - **Practical Insight**: Use apps like Authy or Google Authenticator for 2FA codes.

- **Account Lockdown**:
  - Lock unused accounts: `sudo passwd -l user`.
  - Use no-login shells for service accounts: `sudo usermod -s /sbin/nologin mysql`.
  - Restrict shells for limited users: `sudo usermod -s /bin/rbash user`.
  - **Practical Insight**: Check for active accounts: `getent passwd | grep -v nologin`.

- **Mandatory Access Control (MAC)**:
  - **AppArmor** (Ubuntu/Debian):
    - Install: `sudo apt install apparmor apparmor-profiles`.
    - Enable profile: `sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx`.
    - Verify: `sudo aa-status`.
    - **Practical Insight**: Profiles restrict processes to specific files/permissions (e.g., Nginx can’t access `/home`).
  - **SELinux** (CentOS/RHEL/Fedora):
    - Set enforcing mode: `sudo setenforce 1`.
    - Edit `/etc/selinux/config`:
      ```bash
      SELINUX=enforcing
      ```
    - Check: `getenforce`.
    - Manage: `semanage`, `setsebool` (e.g., `setsebool -P httpd_can_network_connect 1`).
    - **Practical Insight**: Requires learning but provides granular control; start with permissive mode to test.

### 2. **Network Security**

- **Advanced Firewall Rules**:
  - **UFW (Ubuntu)**:
    - Rate-limit SSH: `sudo ufw limit 22`.
    - Allow specific IPs: `sudo ufw allow from 192.168.1.100 to any port 22`.
    - Deny by default: `sudo ufw default deny incoming`.
  - **Firewalld (CentOS/Fedora)**:
    - Create zone: `sudo firewall-cmd --permanent --new-zone=restricted`.
    - Add rules: `sudo firewall-cmd --permanent --zone=restricted --add-source=192.168.1.0/24 --add-service=ssh`.
    - Reload: `sudo firewall-cmd --reload`.
  - **nftables** (Modern, replaces iptables):
    - Edit `/etc/nftables.conf`:
      ```bash
      table inet filter {
          chain input {
              type filter hook input priority 0; policy drop;
              ct state established,related accept
              tcp dport 22 limit rate 10/minute accept
          }
      }
      ```
    - Apply: `sudo systemctl restart nftables`.
  - **Practical Insight**: Test rules with `nmap` or `nc` to ensure only intended ports are open.

- **Fail2Ban for Intrusion Prevention**:
  - Install: `sudo apt install fail2ban`.
  - Configure `/etc/fail2ban/jail.local`:
    ```bash
    [sshd]
    enabled = true
    port = 22
    maxretry = 5
    bantime = 3600
    ```
  - Restart: `sudo systemctl restart fail2ban`.
  - Check: `sudo fail2ban-client status sshd`.
  - **Practical Insight**: Add jails for other services (e.g., `nginx-http-auth`).

- **SSH Hardening**:
  - Edit `/etc/ssh/sshd_config`:
    ```bash
    Port 2222
    PermitRootLogin no
    PasswordAuthentication no
    AllowUsers john alice
    MaxAuthTries 3
    ```
  - Set up key-based authentication:
    - Generate key: `ssh-keygen -t ed25519`.
    - Copy to server: `ssh-copy-id user@server`.
    - Secure keys: `chmod 600 ~/.ssh/id_rsa`.
  - Restart: `sudo systemctl restart sshd`.
  - **Practical Insight**: Use non-standard ports and monitor `/var/log/auth.log` for failed attempts.

- **Network Isolation**:
  - Restrict service access: `sudo nftables -A INPUT -s 192.168.2.0/24 -p tcp --dport 3306 -j ACCEPT`.
  - Use cloud security groups or VLANs for segmentation (e.g., separate web and DB servers).
  - **Practical Insight**: Test connectivity with `nc -zv host port`.

- **Intrusion Detection Systems (IDS)**:
  - **AIDE**:
    - Install: `sudo apt install aide`.
    - Initialize: `sudo aideinit`.
    - Check: `sudo aide --check`.
    - **Practical Insight**: Detects unauthorized file changes (e.g., `/etc/passwd`).
  - **Suricata**:
    - Install: `sudo apt install suricata`.
    - Configure: `/etc/suricata/suricata.yaml`.
    - Run: `sudo suricata -i eth0`.
    - **Practical Insight**: Monitors network for exploits; integrate with SIEM.

### 3. **Service and Application Hardening**

- **Reduce Attack Surface**:
  - List services: `systemctl list-units --type=service --state=running`.
  - Disable unnecessary: `sudo systemctl disable cups`.
  - **Practical Insight**: Use minimal base images (e.g., Ubuntu Server, CentOS Minimal).

- **Web Server Security (Nginx/Apache)**:
  - **Nginx**:
    - Hide version: `server_tokens off` in `/etc/nginx/nginx.conf`.
    - Restrict access: `allow 192.168.1.0/24; deny all;` in server block.
    - Enable TLS: `sudo certbot --nginx`.
  - **Apache**:
    - Hide info: `ServerTokens Prod` and `ServerSignature Off` in `/etc/apache2/conf-available/security.conf`.
    - Enable WAF: `sudo apt install libapache2-mod-security2`.
  - **Practical Insight**: Scan with `nikto` (`sudo apt install nikto`) for vulnerabilities.

- **Database Security (MySQL/MariaDB)**:
  - Secure installation: `sudo mysql_secure_installation`.
  - Bind to localhost: `/etc/mysql/my.cnf`:
    ```bash
    [mysqld]
    bind-address = 127.0.0.1
    ```
  - Use dedicated user: `CREATE USER 'app'@'localhost' IDENTIFIED BY 'strongpassword';`.
  - **Practical Insight**: Encrypt data at rest with `mysql` encryption plugins.

- **Container Security**:
  - Run as non-root: `USER 1000` in `Dockerfile`.
  - Use minimal images: `FROM alpine`.
  - Scan: `docker scan image:tag`.
  - **Practical Insight**: Use `podman` for rootless containers; isolate with `docker network`.

### 4. **Monitoring and Auditing**

- **Centralized Logging**:
  - Configure `rsyslog`:
    - Edit `/etc/rsyslog.conf` to forward logs to a SIEM.
    - Restart: `sudo systemctl restart rsyslog`.
  - Monitor: `tail -f /var/log/syslog` or `journalctl -p 3`.
  - **Practical Insight**: Use log rotation (`/etc/logrotate.conf`) to manage disk space.

- **Auditd**:
  - Install: `sudo apt install auditd`.
  - Add rules: `/etc/audit/audit.rules`:
    ```bash
    -w /etc/passwd -p wa -k passwd_changes
    -a always,exit -F path=/bin/su -F perm=x -k privilege_escalation
    ```
  - Start: `sudo systemctl enable auditd && sudo systemctl start auditd`.
  - Query: `sudo ausearch -k passwd_changes`.
  - **Practical Insight**: Critical for compliance; monitor `/var/log/audit/audit.log`.

- **Lynis**:
  - Install: `sudo apt install lynis`.
  - Run: `sudo lynis audit system`.
  - **Practical Insight**: Follow recommendations to harden system.

- **Wazuh**:
  - Install: Follow Wazuh docs (`curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -`).
  - Configure: `/var/ossec/etc/ossec.conf`.
  - **Practical Insight**: Provides HIDS, FIM, and SIEM integration.

### 5. **Patching and Updates**

- **Automate Security Updates**:
  - **Ubuntu**:
    - Install: `sudo apt install unattended-upgrades`.
    - Configure: `/etc/apt/apt.conf.d/50unattended-upgrades`:
      ```bash
      Unattended-Upgrade::Allowed-Origins {
          "${distro_id}:${distro_codename}-security";
      };
      ```
    - Enable: `sudo dpkg-reconfigure --priority=low unattended-upgrades`.
  - **CentOS/Fedora**:
    - Install: `sudo dnf install dnf-automatic`.
    - Configure: `/etc/dnf/automatic.conf` (`apply_updates = yes`).
    - Enable: `sudo systemctl enable --now dnf-automatic.timer`.
  - **Practical Insight**: Monitor logs (`/var/log/unattended-upgrades`) and test in staging.

- **Vulnerability Management**:
  - Check updates: `sudo apt update && apt list --upgradable` or `sudo dnf updateinfo list security`.
  - Use scanners: `sudo apt install openvas` or `lynis`.
  - **Practical Insight**: Subscribe to distro security advisories (e.g., Ubuntu Security Notices).

- **Kernel Security**:
  - Harden: `/etc/sysctl.conf`:
    ```bash
    net.ipv4.conf.all.accept_redirects = 0
    net.ipv4.tcp_syncookies = 1
    kernel.kptr_restrict = 2
    fs.protected_hardlinks = 1
    fs.protected_symlinks = 1
    ```
  - Apply: `sudo sysctl -p`.
  - **Practical Insight**: Consider `grsecurity` for advanced kernel patches.

### 6. **Incident Response and Recovery**

- **Regular Backups**:
  - Backup: `rsync -av --delete /etc /var/www /backup/`.
  - Schedule: `crontab -e` → `0 2 * * * rsync -av /etc /backup  [TRUNCATED]

System: Apologies, it seems the response was cut off. Let me complete the section and provide the remaining content for **Linux Server Advanced Level Security**, ensuring it’s clear, concise, and practical, as requested. Below is the continuation, starting from the **Incident Response and Recovery** section, followed by practical examples, troubleshooting tips, best practices, practice exercises, and a cheat sheet.


---


## Incident Response and Recovery (Continued)

- **Regular Backups**:
  - Use `rsync` for incremental backups:
    ```bash
    rsync -av --delete /etc /var/www /backup/
    ```
  - Schedule with `cron`: Add to `crontab -e`:
    ```bash
    0 2 * * * rsync -av /etc /backup/ >> /var/log/backup.log 2>&1
    ```
  - Encrypt backups: `tar -czf - /backup | openssl enc -aes-256-cbc -salt -out backup.tar.gz.enc`.
  - Store off-site: Use cloud storage (e.g., AWS S3, encrypted with KMS).
  - **Practical Insight**: Test restores periodically: `openssl dec -aes-256-cbc -in backup.tar.gz.enc | tar -xzf -`.

- **Detecting Compromise**:
  - Monitor processes: `htop` or `ps aux | less` for unusual activity.
  - Check network: `sudo iftop -i eth0` or `tcpdump -i eth0 host suspicious.ip`.
  - Audit logs: `journalctl -p 3` or `sudo ausearch -m LOGIN`.
  - Use rootkit scanners: `sudo apt install chkrootkit rkhunter; sudo rkhunter --check`.
  - **Practical Insight**: Look for unexpected users in `/etc/passwd` or unauthorized SSH keys in `~/.ssh/authorized_keys`.

- **Incident Response Workflow**:
  - **Isolate**: Disconnect network: `sudo ip link set eth0 down`.
  - **Contain**: Kill suspicious processes: `sudo kill -9 <pid>`.
  - **Investigate**: Check logs (`/var/log/auth.log`, `/var/log/syslog`) and AIDE reports.
  - **Recover**: Restore from backup: `rsync -av /backup/etc /etc`.
  - **Document**: Log actions, findings, and lessons learned.
  - **Practical Insight**: Practice in a lab to simulate breaches (e.g., inject a fake malicious script).

## Practical Examples

1. **Harden SSH with 2FA**:
   - Install: `sudo apt install libpam-google-authenticator`.
   - Configure:
     ```bash
     google-authenticator
     sudo nano /etc/pam.d/sshd
     # Add: auth required pam_google_authenticator.so
     sudo nano /etc/ssh/sshd_config
     # Set: ChallengeResponseAuthentication yes
     sudo systemctl restart sshd
     ```
   - Test: Log in with SSH and verify 2FA prompt.

2. **Set Up Fail2Ban for SSH**:
   - Install: `sudo apt install fail2ban`.
   - Configure:
     ```bash
     sudo nano /etc/fail2ban/jail.local
     # Add:
     [sshd]
     enabled = true
     port = 2222
     maxretry = 5
     bantime = 3600
     sudo systemctl restart fail2ban
     ```
   - Verify: `sudo fail2ban-client status sshd`.

3. **Configure AppArmor for a Service**:
   - Install: `sudo apt install apparmor apparmor-profiles`.
   - Enable Nginx profile:
     ```bash
     sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx
     sudo aa-status
     ```
   - Test: Attempt unauthorized access (e.g., `nginx` reading `/home`) and check logs: `journalctl -u apparmor`.

4. **Automate Security Updates**:
   - Ubuntu:
     ```bash
     sudo apt install unattended-upgrades
     sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
     # Ensure security updates are enabled
     sudo dpkg-reconfigure --priority=low unattended-upgrades
     ```
   - CentOS:
     ```bash
     sudo dnf install dnf-automatic
     sudo nano /etc/dnf/automatic.conf
     # Set: apply_updates = yes
     sudo systemctl enable --now dnf-automatic.timer
     ```
   - Verify: `cat /var/log/dnf.log` or `/var/log/unattended-upgrades`.

5. **Audit with Lynis**:
   - Install: `sudo apt install lynis`.
   - Run: `sudo lynis audit system > lynis-report.txt`.
   - Review: `cat lynis-report.txt` and prioritize high-risk findings (e.g., world-writable files).

---

## Troubleshooting Security Issues

1. **SSH Access Issues**:
   - **Symptom**: Cannot log in after hardening.
   - **Fix**:
     - Check `/etc/ssh/sshd_config`: Verify `AllowUsers`, `PermitRootLogin`, `PasswordAuthentication`.
     - Review logs: `journalctl -u sshd`.
     - Test key permissions: `ls -l ~/.ssh/authorized_keys` (should be `600`).
     - Restart: `sudo systemctl restart sshd`.

2. **Firewall Blocking Legitimate Traffic**:
   - **Symptom**: Service (e.g., Nginx) inaccessible.
   - **Fix**:
     - Check rules: `sudo ufw status` or `firewall-cmd --list-all`.
     - Allow port: `sudo ufw allow 80` or `sudo firewall-cmd --add-port=80/tcp --permanent`.
     - Reload: `sudo ufw reload` or `sudo firewall-cmd --reload`.
     - Test: `nc -zv localhost 80`.

3. **Unauthorized File Changes**:
   - **Symptom**: Config files modified unexpectedly.
   - **Fix**:
     - Check audit logs: `sudo ausearch -f /etc/passwd`.
     - Run AIDE: `sudo aide --check`.
     - Restore: `rsync -av /backup/etc /etc`.
     - Investigate: `ps aux | grep <suspicious>` or `lsof -i`.

4. **High Resource Usage Post-Compromise**:
   - **Symptom**: CPU/memory spikes.
   - **Fix**:
     - Identify: `htop` or `top`, sort by CPU/memory.
     - Kill: `sudo kill -9 <pid>`.
     - Scan: `sudo rkhunter --check`.
     - Harden: Restrict user permissions, update software.

5. **SELinux Denials**:
   - **Symptom**: Service fails with permission errors.
   - **Fix**:
     - Check logs: `journalctl -u selinux`.
     - Generate policy: `audit2allow -M mypolicy < /var/log/audit/audit.log`.
     - Apply: `semodule -i mypolicy.pp`.
     - **Note**: Use permissive mode (`setenforce 0`) temporarily to diagnose.

---

## Best Practices

1. **Minimize Attack Surface**:
   - Disable unused services: `sudo systemctl disable telnet`.
   - Remove unnecessary packages: `sudo apt autoremove`.
   - Use minimal OS images (e.g., Ubuntu Server).

2. **Secure Configurations**:
   - Backup configs: `sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak`.
   - Use version control for configs (e.g., Git).
   - Validate syntax: `nginx -t`, `apachectl configtest`.

3. **Proactive Monitoring**:
   - Set up alerts: Use Wazuh or Nagios for real-time notifications.
   - Review logs daily: `journalctl -p 3 -b`.
   - Scan regularly: `lynis audit system`.

4. **Layered Security**:
   - Combine firewalls, AppArmor/SELinux, and IDS.
   - Encrypt sensitive data: Use `gpg` or `openssl` for files.
   - Segregate networks: Isolate web, DB, and admin subnets.

5. **Incident Preparedness**:
   - Maintain an incident response plan: Define roles, contacts, and escalation paths.
   - Conduct tabletop exercises: Simulate a breach.
   - Keep offline backups: Use encrypted external drives.

6. **Compliance and Documentation**:
   - Align with standards: Map controls to PCI-DSS, ISO 27001.
   - Document changes: Log all security configs in a CMS.
   - Audit quarterly: Use Lynis, OpenVAS, or external pentests.

---

## Practice Exercises

1. **Secure SSH with Key-Based Auth and 2FA**:
   - Set up key-based auth: `ssh-keygen -t ed25519; ssh-copy-id user@server`.
   - Enable 2FA (see example above).
   - Test: Log in and verify key + 2FA requirement.
   - Check logs: `tail -f /var/log/auth.log`.

2. **Configure a Secure Web Server**:
   - Install Nginx: `sudo apt install nginx`.
   - Harden: `sudo nano /etc/nginx/nginx.conf` (set `server_tokens off`, add TLS).
   - Set permissions: `sudo chown -R www-data:www-data /var/www/html; sudo chmod -R 755 /var/www/html`.
   - Test: `curl -I https://localhost`.

3. **Set Up Auditd for Compliance**:
   - Install: `sudo apt install auditd`.
   - Add rule: `sudo nano /etc/audit/audit.rules` (monitor `/etc/shadow`).
   - Start: `sudo systemctl enable auditd && sudo systemctl start auditd`.
   - Query: `sudo ausearch -f /etc/shadow`.

4. **Automate Backups and Test Restore**:
   - Script backup: `nano backup.sh` with `rsync -av /etc /backup/`.
   - Schedule: `crontab -e` (add `0 2 * * * /path/to/backup.sh`).
   - Test restore: `rsync -av /backup/etc /tmp/test-restore`.
   - Verify: `diff -r /etc /tmp/test-restore`.

5. **Simulate and Respond to a Breach**:
   - Create a fake malicious script: `echo 'while true; do echo consuming cpu; done' > /tmp/malware.sh; chmod +x /tmp/malware.sh`.
   - Run: `bash /tmp/malware.sh &`.
   - Detect: `htop` (find high CPU process).
   - Respond: `kill <pid>`, remove script, check logs: `journalctl -p 3`.

## Cheat Sheet

| Task                          | Command Example                                      |
|-------------------------------|-----------------------------------------------------|
| Disable root SSH              | `sudo nano /etc/ssh/sshd_config; PermitRootLogin no` |
| Enforce password policy       | `sudo nano /etc/security/pwquality.conf`            |
| Enable 2FA for SSH            | `auth required pam_google_authenticator.so`         |
| Lock account                  | `sudo passwd -l user`                               |
| Enable AppArmor profile       | `sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx`    |
| Set SELinux enforcing         | `sudo setenforce 1`                                 |
| Rate-limit SSH (UFW)          | `sudo ufw limit 22`                                 |
| Configure Fail2Ban            | `sudo nano /etc/fail2ban/jail.local`                |
| Harden Nginx                  | `server_tokens off` in `/etc/nginx/nginx.conf`      |
| Secure MySQL                  | `sudo mysql_secure_installation`                    |
| Monitor logs                  | `journalctl -p 3`                                   |
| Set up Auditd                 | `sudo nano /etc/audit/audit.rules`                  |
| Run Lynis audit               | `sudo lynis audit system`                           |
| Automate updates (Ubuntu)     | `sudo dpkg-reconfigure unattended-upgrades`         |
| Harden kernel                 | `sudo sysctl -p /etc/sysctl.conf`                   |
| Backup data                   | `rsync -av /etc /backup/`                           |
| Detect rootkits               | `sudo rkhunter --check`                             |

## Resources

- **Man Pages**: `man sshd_config`, `man auditd`, `man sysctl`, `man rsync`.
- **Online**: 
  - Linux Security Guides: Linux Journey, Arch Wiki, CIS Benchmarks.
  - Tools: Lynis (`lynis.io`), Wazuh (`wazuh.com`), OpenVAS (`openvas.org`).
- **Courses**: SANS SEC560, CompTIA Security+, Linux Foundation Security.
- **Practice**: Set up a VM (e.g., VirtualBox with Ubuntu Server) and apply hardening steps.

---

### Next Steps
- Practice these security measures on a Linux VM (e.g., secure SSH, set up a firewall, audit with Lynis).
- Share the next topic (e.g., “Logging and Monitoring,” “Container Security,” or “Cloud Security”) or ask for clarification, and I’ll provide clear, practical notes.
- Want a challenge? Try scenarios like “Harden a LAMP stack for production” or “Simulate a brute-force attack and mitigate it.”

---