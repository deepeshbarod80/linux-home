
# Linux System Services

In Linux, **services** are background processes (daemons) that perform specific functions, such as running a web server, managing SSH connections, or logging system events. Services are managed by an **init system**, which initializes and controls them during system boot and runtime. Mastering service management is essential for configuring servers, ensuring uptime, and troubleshooting issues.

## Key Concepts

- **Service (Daemon)**: A program running in the background, typically with a name ending in `d` (e.g., `sshd` for SSH, `httpd` for Apache).
- **Init System**: The system that manages services, starting them at boot and handling their lifecycle.
- **Common Init Systems**:
  - **systemd**: Modern, widely used (Ubuntu, CentOS, Fedora).
  - **SysVinit**: Older, script-based (used in legacy systems).
  - **OpenRC**: Lightweight (used in Gentoo, some Alpine setups).
- **Service States**:
  - **Running/Active**: The service is operational.
  - **Stopped/Inactive**: The service is not running.
  - **Enabled**: Starts automatically at boot.
  - **Disabled**: Does not start at boot.
- **Unit Files**: Configuration files (for systemd) defining how services run (e.g., `/lib/systemd/system/nginx.service`).
- **Practical Use**: Service management is critical for tasks like starting a web server, enabling SSH, or scheduling automated tasks.

## Init Systems Overview

### 1. **systemd** (Most Common)
- **Used By**: Ubuntu, Debian, CentOS, Fedora, RHEL, and most modern distros.
- **Key Features**:
  - Manages services, mounts, sockets, and more via **units** (e.g., `.service`, `.timer`).
  - Parallelizes service startup for faster boot.
  - Provides detailed logging via `journalctl`.
- **Configuration**: Service files in `/lib/systemd/system/` (system) or `/etc/systemd/system/` (custom).
- **Tools**: `systemctl` (manage services), `journalctl` (view logs).

### 2. **SysVinit** (Legacy)
- **Used By**: Older distros (e.g., CentOS 6, Debian 7).
- **Key Features**:
  - Uses shell scripts in `/etc/init.d/` to manage services.
  - Runlevels (e.g., 3 for multi-user, 5 for GUI) determine which services start.
- **Tools**: `service`, `/etc/init.d/<service>`, `chkconfig` (for enabling/disabling).

### 3. **OpenRC**
- **Used By**: Gentoo, some lightweight distros.
- **Key Features**:
  - Simple, script-based, and modular.
  - Uses `/etc/init.d/` for scripts and `/etc/conf.d/` for configs.
- **Tools**: `rc-service`, `rc-update`.

**Focus**: Since **systemd** is the most widely used init system, these notes will primarily cover `systemd` with brief mentions of SysVinit for context. If you need OpenRC or SysVinit specifics, let me know!

## Managing Services with `systemd`

The `systemctl` command is the primary tool for managing services in systemd. Below are the key commands, options, and practical examples.

### 1. **Common `systemctl` Commands**

- **`systemctl status <service>`**:
  - Purpose: Shows the status of a service (running, stopped, errors).
  - Example: `systemctl status nginx` → Displays Nginx status, recent logs, and PID.
  - Practical Insight: Use to troubleshoot (e.g., check why a service failed).

- **`systemctl start <service>`**:
  - Purpose: Starts a service immediately.
  - Example: `sudo systemctl start sshd` → Starts the SSH daemon.
  - Practical Insight: Does not affect boot behavior.

- **`systemctl stop <service>`**:
  - Purpose: Stops a running service.
  - Example: `sudo systemctl stop apache2` → Stops the Apache web server.
  - Practical Insight: Use to free resources or troubleshoot.

- **`systemctl restart <service>`**:
  - Purpose: Stops and starts a service (useful after config changes).
  - Example: `sudo systemctl restart nginx` → Restarts Nginx to apply new settings.
  - Practical Insight: Safer than `reload` for major changes.

- **`systemctl reload <service>`**:
  - Purpose: Reloads a service’s configuration without stopping it.
  - Example: `sudo systemctl reload nginx` → Applies new Nginx config without downtime.
  - Practical Insight: Use for services like web servers to avoid interrupting users.

- **`systemctl enable <service>`**:
  - Purpose: Configures a service to start automatically at boot.
  - Example: `sudo systemctl enable mariadb` → Enables the MariaDB database at boot.
  - Practical Insight: Creates symlinks in `/etc/systemd/system/`.

- **`systemctl disable <service>`**:
  - Purpose: Prevents a service from starting at boot.
  - Example: `sudo systemctl disable bluetooth` → Disables Bluetooth at boot.
  - Practical Insight: Does not stop the service if it’s currently running.

- **`systemctl list-units --type=service`**:
  - Purpose: Lists all loaded services and their status.
  - Example: `systemctl list-units --type=service --state=running` → Shows running services.
  - Practical Insight: Use to explore active services on a server.

- **`systemctl list-unit-files --type=service`**:
  - Purpose: Lists all service unit files and their enabled/disabled state.
  - Example: `systemctl list-unit-files --type=service | grep enabled` → Shows enabled services.
  - Practical Insight: Helps audit boot-time services.

### 2. **Viewing Service Logs with `journalctl`**

- **Purpose**: Displays logs for services and the system (systemd’s logging tool).
- **Key Commands**:
  - View all logs: `journalctl`
  - View service logs: `journalctl -u nginx` → Shows Nginx logs.
  - Follow logs in real-time: `journalctl -u sshd -f` → Like `tail -f`.
  - Filter by time: `journalctl -u apache2 --since "2025-04-20"`
  - Show errors: `journalctl -p 3` → Displays error-level logs.
- **Practical Insight**:
  - Use `-x` for detailed explanations: `journalctl -u mysql -x`.
  - Check recent logs for troubleshooting: `journalctl -u <service> -n 50`.
  - Logs are stored in memory or `/var/log/journal/` (if persistent).

### 3. **Managing Service Dependencies**

- **Purpose**: Services often depend on others (e.g., a web server needs networking).
- **Unit File Keywords**:
  - `Wants`: Optional dependencies.
  - `Requires`: Mandatory dependencies.
  - `After`: Specifies start order.
- **Example**: In `/lib/systemd/system/nginx.service`:
  ```ini
  [Unit]
  Requires=network.target
  After=network.target
  ```
  Ensures Nginx starts after networking is available.
- **Practical Insight**:
  - Check dependencies: `systemctl list-dependencies nginx`.
  - Edit custom configs in `/etc/systemd/system/` (not `/lib/systemd/system/`).

### 4. **Creating/Modifying Services**

- **Purpose**: Define custom services for scripts or applications.
- **Steps**:
  1. Create a service file: `sudo nano /etc/systemd/system/myservice.service`.
  2. Add content (example):
     ```ini
     [Unit]
     Description=My Custom Service
     After=network.target

     [Service]
     ExecStart=/usr/bin/myscript.sh
     Restart=always

     [Install]
     WantedBy=multi-user.target
     ```
  3. Reload systemd: `sudo systemctl daemon-reload`.
  4. Enable and start: `sudo systemctl enable myservice && sudo systemctl start myservice`.
- **Practical Insight**:
  - Use for custom apps or cron-like tasks.
  - Test with `systemctl status myservice`.

## Managing Services with SysVinit (Legacy)

For older systems using SysVinit, service management is script-based.

- **Key Commands**:
  - Start: `sudo /etc/init.d/<service> start` (e.g., `sudo /etc/init.d/apache2 start`).
  - Stop: `sudo /etc/init.d/<service> stop`.
  - Status: `sudo /etc/init.d/<service> status`.
  - Enable at boot: `sudo chkconfig <service> on` (or `update-rc.d` on Debian).
- **Alternative**: Use `service` command (works with both SysVinit and systemd):
  - `sudo service ssh start`
  - `sudo service nginx status`
- **Practical Insight**:
  - Check scripts in `/etc/init.d/`.
  - Less flexible than systemd; logs are typically in `/var/log/`.

## Common Services and Their Use Cases

| Service       | Purpose                          | Example Commands                              |
|---------------|----------------------------------|-----------------------------------------------|
| `sshd`        | SSH server for remote access     | `sudo systemctl restart sshd`                 |
| `nginx`/`httpd` | Web server (Nginx/Apache)       | `sudo systemctl reload nginx`                 |
| `mariadb`/`mysql` | Database server               | `sudo systemctl enable mariadb`               |
| `cron`        | Scheduled tasks                  | `sudo systemctl status cron`                  |
| `network`     | Network management               | `sudo systemctl restart networking` (SysVinit)|
| `ufw`/`firewalld` | Firewall                     | `sudo systemctl enable ufw`                   |

## Practical Tips for Service Management

1. **Hands-On Practice**:
   - Install a service: `sudo apt install nginx` (Ubuntu) or `sudo dnf install httpd` (Fedora).
   - Start and check: `sudo systemctl start nginx && systemctl status nginx`.
   - Enable at boot: `sudo systemctl enable nginx`.
   - View logs: `journalctl -u nginx -n 50`.

2. **Troubleshooting**:
   - **Service Won’t Start**:
     - Check status: `systemctl status <service>` (look for errors).
     - View logs: `journalctl -u <service> -xe`.
     - Verify configs (e.g., `nginx -t` for Nginx syntax check).
   - **Permission Issues**:
     - Ensure correct ownership (e.g., `sudo chown www-data:www-data /var/www/html`).
     - Check permissions: `ls -l /var/log/nginx`.
   - **Port Conflicts**:
     - Check listening ports: `sudo netstat -tuln` or `ss -tuln`.
     - Kill conflicting processes: `sudo kill <pid>`.

3. **Best Practices**:
   - **Minimize Running Services**: Disable unnecessary services to reduce attack surface: `sudo systemctl disable bluetooth`.
   - **Test Config Changes**: Use `reload` instead of `restart` to avoid downtime (e.g., `sudo systemctl reload nginx`).
   - **Monitor Logs**: Regularly check `journalctl` for errors or warnings.
   - **Backup Configs**: Before editing service files, back up: `sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak`.
   - **Security**: Ensure services like `sshd` use secure configs (e.g., disable root login in `/etc/ssh/sshd_config`).

4. **Automation**:
   - Use `systemd` timers instead of `cron` for modern systems (e.g., create a `.timer` unit).
   - Script service checks: `systemctl is-active nginx || systemctl start nginx`.

5. **Server Use Cases**:
   - **Web Server**: Enable and start `nginx` or `httpd`, ensure port 80/443 is open.
   - **Database**: Secure `mysql` with proper permissions on `/var/lib/mysql`.
   - **SSH**: Restrict `sshd` to specific users and enable at boot.

## Practice Exercises

1. **Explore Services**:
   - List running services: `systemctl list-units --type=service --state=running`.
   - Check SSH status: `systemctl status sshd`.
   - View recent logs: `journalctl -u sshd -n 20`.

2. **Manage a Web Server**:
   - Install Nginx: `sudo apt install nginx` (Ubuntu) or `sudo dnf install nginx` (Fedora).
   - Start and enable: `sudo systemctl start nginx && sudo systemctl enable nginx`.
   - Reload after config change: `sudo nano /etc/nginx/nginx.conf`, then `sudo systemctl reload nginx`.
   - Check logs: `journalctl -u nginx`.

3. **Create a Custom Service**:
   - Create a script: `echo -e '#!/bin/bash\necho "Running" >> /tmp/myservice.log' | sudo tee /usr/bin/myscript.sh`.
   - Make executable: `sudo chmod +x /usr/bin/myscript.sh`.
   - Create a service file: `sudo nano /etc/systemd/system/myservice.service` with the example above.
   - Enable and start: `sudo systemctl daemon-reload && sudo systemctl enable myservice && sudo systemctl start myservice`.
   - Verify: `cat /tmp/myservice.log`.

4. **Troubleshoot a Service**:
   - Stop a service: `sudo systemctl stop nginx`.
   - Try starting with an invalid config (edit `/etc/nginx/nginx.conf` to introduce a syntax error).
   - Check status: `systemctl status nginx` and logs: `journalctl -u nginx -xe`.
   - Fix and restart: `sudo nginx -t && sudo systemctl restart nginx`.

5. **Disable Unnecessary Services**:
   - List enabled services: `systemctl list-unit-files --type=service | grep enabled`.
   - Disable one: `sudo systemctl disable cups` (printing service, if unused).
   - Verify: `systemctl is-enabled cups`.

## Cheat Sheet

| Task                     | Command Example                              |
|--------------------------|----------------------------------------------|
| Check service status     | `systemctl status nginx`                     |
| Start service            | `sudo systemctl start nginx`                 |
| Stop service             | `sudo systemctl stop nginx`                  |
| Restart service          | `sudo systemctl restart nginx`               |
| Reload service           | `sudo systemctl reload nginx`                |
| Enable at boot           | `sudo systemctl enable nginx`                |
| Disable at boot          | `sudo systemctl disable nginx`               |
| List running services    | `systemctl list-units --type=service`        |
| View service logs        | `journalctl -u nginx`                        |
| Follow logs in real-time | `journalctl -u nginx -f`                     |
| Reload systemd config    | `sudo systemctl daemon-reload`               |
| SysVinit start (legacy)  | `sudo /etc/init.d/apache2 start`             |
| SysVinit enable (legacy) | `sudo chkconfig apache2 on`                  |

## Resources

- **Man Pages**: `man systemctl`, `man journalctl`, `man systemd.service`.
- **Online**: Systemd documentation, Arch Wiki (`systemd` section), Ubuntu Server Guide.
- **Practice**: Set up a VM and experiment with services like `nginx`, `mariadb`, or `cron`.

---
