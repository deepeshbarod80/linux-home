
# Logging and Monitoring in Linux Servers

**Logging** involves recording system and application events (e.g., errors, login attempts, service activity) for analysis, while **monitoring** tracks real-time performance metrics (e.g., CPU, memory, network) and alerts on anomalies. Together, they enable proactive issue detection, performance optimization, and security incident response on Linux servers.

## Key Concepts

- **Logs**: Text files or structured data capturing events (e.g., `/var/log/syslog`, `/var/log/auth.log`).
- **Log Types**:
  - **System Logs**: Kernel, boot, and service events.
  - **Application Logs**: Web servers (e.g., Nginx), databases (e.g., MySQL).
  - **Security Logs**: Authentication, sudo, and firewall events.
- **Monitoring**: Real-time or periodic collection of metrics (e.g., CPU usage, disk I/O, network traffic).
- **Log Management**: Collecting, rotating, and analyzing logs to prevent disk space issues and enable auditing.
- **Monitoring Tools**: Track resource usage, service health, and trigger alerts.
- **Centralized Logging**: Aggregating logs from multiple servers for easier analysis (e.g., ELK Stack, Graylog).
- **Practical Use**: Logging and monitoring are essential for troubleshooting (e.g., why a service crashed), security (e.g., detecting brute-force attacks), and compliance (e.g., SOC 2, HIPAA).

## Core Logging Components

1. **Log Files** (Typically in `/var/log/`):
   - `/var/log/syslog` or `/var/log/messages`: General system and application logs (Ubuntu vs. CentOS).
   - `/var/log/auth.log` or `/var/log/secure`: Authentication and sudo events.
   - `/var/log/kern.log`: Kernel messages.
   - `/var/log/dpkg.log` or `/var/log/yum.log`: Package manager logs.
   - Application-specific: `/var/log/nginx/access.log`, `/var/log/mysql/error.log`.

2. **Logging Daemons**:
   - **rsyslog**: Default on most distros; collects and forwards logs.
   - **systemd-journald**: Manages logs for systemd services; stores in binary format.
   - **auditd**: Tracks security-related events (e.g., file access, sudo usage).

3. **Log Rotation**:
   - Managed by `logrotate` (`/etc/logrotate.conf`, `/etc/logrotate.d/`).
   - Prevents logs from consuming disk space.

## Core Monitoring Components

1. **System Metrics**:
   - CPU, memory, disk, and network usage.
   - Tools: `top`, `htop`, `vmstat`, `iostat`.

2. **Service Health**:
   - Check service status: `systemctl status <service>`.
   - Monitor ports: `ss -tuln`.

3. **Advanced Monitoring**:
   - Tools: Prometheus, Nagios, Zabbix.
   - Alerting: Notify on thresholds (e.g., CPU > 90%).

## Essential Logging and Monitoring Commands and Tools

Below are the core tools and commands for logging and monitoring, with explanations, options, and practical examples.

### 1. **Logging Tools**

- **`journalctl` (systemd-journald)**:
  - Purpose: Queries systemd logs (binary format, no need for `/var/log` files).
  - Common Options:
    - `-u <service>`: Filter by service (e.g., `nginx`).
    - `-p <level>`: Filter by priority (e.g., `3` for errors).
    - `-f`: Follow logs in real-time (like `tail -f`).
    - `--since`: Filter by time (e.g., `"2025-04-20 10:00"`).
    - `-n <num>`: Show last N lines.
  - Examples:
    - `journalctl -u sshd` → Show SSH service logs.
    - `journalctl -p 3` → Display error-level logs.
    - `journalctl -f` → Follow logs in real-time.
    - `journalctl --since "yesterday"` → Show logs since yesterday.
    - `journalctl -u nginx -n 50` → Last 50 lines of Nginx logs.
  - **Practical Insight**:
    - Use `-x` for detailed explanations: `journalctl -u mysql -x`.
    - Persist logs: Edit `/etc/systemd/journald.conf` to set `Storage=persistent`.

- **`rsyslog`**:
  - Purpose: Collects and forwards logs to files or remote servers.
  - Configuration: `/etc/rsyslog.conf`, `/etc/rsyslog.d/`.
  - Example Config (`/etc/rsyslog.d/remote.conf`):
    ```bash
    *.* @192.168.1.200:514  # Forward all logs to remote syslog server (UDP)
    *.* @@192.168.1.200:514 # Use TCP
    ```
  - Restart: `sudo systemctl restart rsyslog`.
  - View Logs: `tail -f /var/log/syslog` or `/var/log/messages`.
  - **Practical Insight**: Use for centralized logging with tools like ELK Stack.

- **`logrotate`**:
  - Purpose: Rotates, compresses, and deletes old logs to manage disk space.
  - Configuration: `/etc/logrotate.conf`, `/etc/logrotate.d/`.
  - Example (`/etc/logrotate.d/nginx`):
    ```bash
    /var/log/nginx/*.log {
        daily
        rotate 7
        compress
        delaycompress
        missingok
        notifempty
        create 0640 www-data adm
    }
    ```
  - Test: `sudo logrotate -f /etc/logrotate.conf`.
  - **Practical Insight**: Check disk usage: `du -sh /var/log/*`.

- **`auditd`**:
  - Purpose: Tracks security events (e.g., file access, sudo commands).
  - Install: `sudo apt install auditd audispd-plugins`.
  - Configure: `/etc/audit/audit.rules`:
    ```bash
    -w /etc/passwd -p wa -k passwd_changes
    -a always,exit -F path=/bin/su -F perm=x -k privilege_escalation
    ```
  - Start: `sudo systemctl enable auditd && sudo systemctl start auditd`.
  - Query: `sudo ausearch -k passwd_changes`.
  - Logs: `/var/log/audit/audit.log`.
  - **Practical Insight**: Essential for compliance (e.g., PCI-DSS, SOC 2).

### 2. **Basic Monitoring Tools**

- **`top` and `htop`**:
  - Purpose: Real-time process and resource monitoring.
  - Examples:
    - `top` → Sort by CPU (`P`), memory (`M`); press `q` to quit.
    - `htop` → Interactive, colorized (install: `sudo apt install htop`).
  - **Practical Insight**: Use to identify high CPU/memory processes.

- **`free`**:
  - Purpose: Displays memory usage.
  - Example: `free -h` → Human-readable memory stats.
  - **Practical Insight**: Check for low free memory or high swap usage.

- **`df`**:
  - Purpose: Shows disk usage.
  - Example: `df -h` → Human-readable disk stats.
  - **Practical Insight**: Monitor `/var/log` to prevent disk exhaustion.

- **`iostat`**:
  - Purpose: Monitors CPU and disk I/O (requires `sysstat`).
  - Example: `iostat -dx 1` → Show disk I/O every second.
  - **Practical Insight**: Identify I/O bottlenecks affecting services.

- **`vmstat`**:
  - Purpose: Reports virtual memory, CPU, and I/O stats.
  - Example: `vmstat 1` → Update every second.
  - **Practical Insight**: Watch `si/so` for swap activity (high values indicate memory pressure).

- **`ss`**:
  - Purpose: Monitors network sockets.
  - Example: `ss -tuln` → List listening ports.
  - **Practical Insight**: Use to verify service connectivity (e.g., port 80 for Nginx).

### 3. **Advanced Monitoring Tools**

- **Prometheus and Grafana**:
  - **Prometheus**: Time-series database for metrics collection.
    - Install: Download from `prometheus.io` or `sudo apt install prometheus`.
    - Configure: `/etc/prometheus/prometheus.yml`:
      ```yaml
      scrape_configs:
        - job_name: 'node'
          static_configs:
            - targets: ['localhost:9100']
      ```
    - Start: `sudo systemctl enable prometheus && sudo systemctl start prometheus`.
  - **Node Exporter**: Collects system metrics (CPU, memory, disk).
    - Install: `sudo apt install prometheus-node-exporter`.
    - Start: `sudo systemctl enable prometheus-node-exporter`.
  - **Grafana**: Visualizes Prometheus data.
    - Install: `sudo apt install grafana`.
    - Configure: Access `http://server:3000`, add Prometheus data source.
    - Create dashboards: Import pre-built dashboards (e.g., Node Exporter Full).
  - **Practical Insight**: Set up alerts in Prometheus (`alertmanager`) for CPU > 90% or disk > 80%.

- **Zabbix**:
  - Purpose: Comprehensive monitoring with agent-based and agentless options.
  - Install: Follow `zabbix.com` docs (requires server, database, frontend).
  - Configure: Add hosts, set triggers (e.g., alert on failed SSH logins).
  - **Practical Insight**: Ideal for large environments; supports auto-discovery.

- **Nagios**:
  - Purpose: Classic monitoring with plugins for services and metrics.
  - Install: `sudo apt install nagios4`.
  - Configure: `/etc/nagios4/nagios.cfg`, `/etc/nagios4/objects/`.
  - **Practical Insight**: Use for simple setups; extend with plugins (e.g., `check_disk`).

### 4. **Centralized Logging**

- **ELK Stack (Elasticsearch, Logstash, Kibana)**:
  - **Elasticsearch**: Stores and indexes logs.
  - **Logstash**: Processes and forwards logs.
  - **Kibana**: Visualizes logs via dashboards.
  - Install: Follow `elastic.co` docs (or use Docker: `docker-compose.yml`).
  - Configure Logstash (`/etc/logstash/conf.d/syslog.conf`):
    ```bash
    input { syslog { port => 514 } }
    output { elasticsearch { hosts => ["localhost:9200"] } }
    ```
  - Access Kibana: `http://server:5601`.
  - **Practical Insight**: Use Filebeat (`sudo apt install filebeat`) to ship logs to ELK.

- **Graylog**:
  - Purpose: Alternative to ELK with simpler setup.
  - Install: Follow `graylog.org` docs (requires Elasticsearch, MongoDB).
  - Configure: Add inputs (e.g., Syslog UDP on port 514).
  - **Practical Insight**: Lightweight for small-to-medium setups.

## Practical Examples

1. **Monitor SSH Login Attempts**:
   - View logs: `journalctl -u sshd | grep "Failed password"`.
   - Audit logins: `sudo ausearch -m USER_LOGIN`.
   - Set up Fail2Ban (from security topic) to block brute-force attempts.

2. **Rotate Nginx Logs**:
   - Create config: `sudo nano /etc/logrotate.d/nginx` (use example above).
   - Test: `sudo logrotate -f /etc/logrotate.d/nginx`.
   - Verify: `ls -l /var/log/nginx/*.gz`.

3. **Set Up Prometheus and Grafana**:
   - Install: `sudo apt install prometheus prometheus-node-exporter grafana`.
   - Configure Prometheus: Add node exporter target in `/etc/prometheus/prometheus.yml`.
   - Start services: `sudo systemctl enable prometheus prometheus-node-exporter grafana-server`.
   - Access Grafana: `http://server:3000`, add Prometheus data source, import dashboard ID 1860.
   - Check metrics: `curl http://localhost:9100/metrics`.

4. **Centralize Logs with rsyslog**:
   - Configure: `sudo nano /etc/rsyslog.d/remote.conf` (forward to `192.168.1.200:514`).
   - Restart: `sudo systemctl restart rsyslog`.
   - Verify: On remote server, check `/var/log/syslog` or use `tcpdump -i eth0 port 514`.

5. **Audit File Access**:
   - Install auditd: `sudo apt install auditd`.
   - Add rule: `sudo nano /etc/audit/audit.rules` (monitor `/etc/shadow`).
   - Query: `sudo ausearch -f /etc/shadow`.
   - Check logs: `tail -f /var/log/audit/audit.log`.

## Troubleshooting Logging and Monitoring Issues

1. **Logs Not Appearing**:
   - Check service: `sudo systemctl status rsyslog` or `journalctl -u systemd-journald`.
   - Verify config: `sudo rsyslogd -N1` (test rsyslog syntax).
   - Ensure permissions: `ls -l /var/log/syslog` (should be writable by `syslog`).

2. **Disk Space Issues**:
   - Check usage: `du -sh /var/log/*`.
   - Rotate logs: `sudo logrotate -f /etc/logrotate.conf`.
   - Clear old logs: `sudo find /var/log -type f -name "*.gz" -delete`.

3. **Monitoring Alerts Not Triggering**:
   - Verify Prometheus rules: `cat /etc/prometheus/rules.yml`.
   - Check Alertmanager: `curl http://localhost:9093/api/v1/alerts`.
   - Test metric: `curl http://localhost:9100/metrics | grep node_cpu`.

4. **Centralized Logging Failure**:
   - Check network: `nc -zv 192.168.1.200 514`.
   - Verify remote server: `tail -f /var/log/syslog` on remote.
   - Debug Logstash: `sudo /usr/share/logstash/bin/logstash --config.test_and_exit -f /etc/logstash/conf.d/`.

5. **Auditd Overload**:
   - Check disk: `df -h /var/log/audit`.
   - Reduce rules: Edit `/etc/audit/audit.rules` to focus on critical files.
   - Increase buffer: `sudo nano /etc/audit/auditd.conf`, set `max_log_file = 100`.

## Best Practices

1. **Centralize Logs**:
   - Use ELK, Graylog, or rsyslog for multi-server environments.
   - Encrypt log transmission: Use TLS for rsyslog (`@@server:514`).

2. **Monitor Proactively**:
   - Set thresholds: Alert on CPU > 90%, disk > 80%.
   - Use dashboards: Grafana for visualization.
   - Automate checks: Script `systemctl is-active nginx` with cron.

3. **Secure Logs**:
   - Restrict access: `sudo chmod 640 /var/log/syslog; sudo chown syslog:adm /var/log/syslog`.
   - Monitor log tampering: Use `auditd` to track `/var/log`.
   - Backup logs: `rsync -av /var/log /backup/logs`.

4. **Optimize Log Storage**:
   - Configure `logrotate`: Daily rotation, compress after 7 days.
   - Monitor `/var/log`: `df -h /var/log` weekly.
   - Use external storage for long-term logs.

5. **Compliance**:
   - Retain logs for required periods (e.g., 1 year for PCI-DSS).
   - Use `auditd` for security event tracking.
   - Document monitoring policies: Define metrics, alerts, and escalation paths.

6. **Test and Validate**:
   - Simulate failures: Stop a service (`sudo systemctl stop nginx`) and check alerts.
   - Test log forwarding: Trigger an event (e.g., failed SSH login) and verify remote logs.
   - Review dashboards: Ensure Grafana shows accurate metrics.

## Practice Exercises

1. **Explore System Logs**:
   - View recent errors: `journalctl -p 3 -n 50`.
   - Monitor SSH: `journalctl -u sshd -f`.
   - Check authentication: `tail -f /var/log/auth.log`.

2. **Configure Log Rotation**:
   - Create: `sudo nano /etc/logrotate.d/testlog`:
     ```bash
     /var/log/test.log {
         daily
         rotate 7
         compress
         missingok
     }
     ```
   - Test: `sudo logrotate -f /etc/logrotate.d/testlog`.
   - Verify: `ls -l /var/log/test.log*`.

3. **Set Up Prometheus and Grafana**:
   - Install: `sudo apt install prometheus prometheus-node-exporter grafana`.
   - Configure: Add node exporter to `/etc/prometheus/prometheus.yml`.
   - Start: `sudo systemctl enable prometheus grafana-server`.
   - Create dashboard: Log into Grafana (`http://server:3000`), import dashboard 1860.

4. **Centralize Logs with rsyslog**:
   - Configure: `sudo nano /etc/rsyslog.d/remote.conf` (forward to a remote server).
   - Restart: `sudo systemctl restart rsyslog`.
   - Test: Trigger a log (`logger "Test message"`) and check remote server.

5. **Audit sudo Usage**:
   - Install: `sudo apt install auditd`.
   - Add rule: `sudo nano /etc/audit/audit.rules` (monitor `/bin/sudo`).
   - Test: Run `sudo ls` and query: `sudo ausearch -m EXECVE -k privilege_escalation`.

## Cheat Sheet

| Task                          | Command Example                                      |
|-------------------------------|-----------------------------------------------------|
| View service logs             | `journalctl -u nginx`                               |
| Follow logs in real-time      | `journalctl -f`                                     |
| Filter errors                 | `journalctl -p 3`                                   |
| View logs since time          | `journalctl --since "2025-04-20"`                   |
| Configure rsyslog             | `sudo nano /etc/rsyslog.d/remote.conf`              |
| Rotate logs                   | `sudo logrotate -f /etc/logrotate.conf`             |
| Set up auditd                 | `sudo nano /etc/audit/audit.rules`                  |
| Query audit logs              | `sudo ausearch -k passwd_changes`                   |
| Monitor processes             | `htop` or `top`                                     |
| Check memory                  | `free -h`                                           |
| Monitor disk usage            | `df -h`                                             |
| Monitor I/O                   | `iostat -dx 1`                                      |
| Check open ports              | `ss -tuln`                                          |
| Install Prometheus            | `sudo apt install prometheus`                       |
| Access Grafana                | `http://server:3000`                                |
| Configure ELK input           | `sudo nano /etc/logstash/conf.d/syslog.conf`        |

## Resources

- **Man Pages**: `man journalctl`, `man rsyslogd`, `man logrotate`, `man auditd`.
- **Online**:
  - Logging: Arch Wiki (`rsyslog`, `systemd-journald`), Elastic Docs (`elastic.co`).
  - Monitoring: Prometheus (`prometheus.io`), Grafana (`grafana.com`), Zabbix (`zabbix.com`).
- **Practice**: Set up a VM (e.g., Ubuntu Server) and install Prometheus/Grafana, configure rsyslog, and test auditd.

---
