
# Linux User Management

In Linux, **user management** involves creating, modifying, and deleting user accounts and groups to control access to the system and its resources. Each user has a unique identity, permissions, and home directory, while groups allow shared access to files and resources. Proper user management is critical for security, collaboration, and system administration on Linux servers.

## Key Concepts

- **User**: An account representing a person or service (e.g., `john`, `root`, `www-data`).
- **Group**: A collection of users for shared permissions (e.g., `sudo`, `developers`).
- **User Types**:
  - **Root**: Superuser with full system access (UID 0).
  - **Regular Users**: Standard accounts for individuals (UID ≥ 1000 in most distros).
  - **System Users**: Accounts for services/daemons (e.g., `mysql`, `nginx`; UID 100–999).
- **User ID (UID)**: Unique numeric identifier for a user.
- **Group ID (GID)**: Unique numeric identifier for a group.
- **Key Files**:
  - `/etc/passwd`: Stores user account information (username, UID, GID, home directory, shell).
  - `/etc/shadow`: Stores encrypted passwords and password policies.
  - `/etc/group`: Defines groups and their members.
  - `/etc/gshadow`: Stores group passwords (rarely used).
- **Home Directory**: Default storage for user files (e.g., `/home/john`).
- **Shell**: User’s default command-line interface (e.g., `/bin/bash`, `/bin/zsh`).
- **Practical Use**: User management ensures secure access, restricts permissions, and supports collaboration (e.g., shared project directories).

## Essential User Management Commands

Below are the core commands for managing users and groups, with explanations, options, and practical examples.

### 1. **Viewing User and Group Information**

- **`whoami`**:
  - Purpose: Displays the current user’s username.
  - Example: `whoami` → Output: `john`.
  - Practical Insight: Quick way to confirm your identity.

- **`id`**:
  - Purpose: Shows the current user’s UID, GID, and group memberships.
  - Options:
    - `-u`: Show UID only.
    - `-g`: Show primary GID.
    - `-G`: Show all group IDs.
  - Example: `id` → Output: `uid=1000(john) gid=1000(john) groups=1000(john),27(sudo)`.
  - Practical Insight: Use to verify group memberships for permission issues.

- **`cat /etc/passwd`**:
  - Purpose: Displays user account details.
  - Example: `cat /etc/passwd | grep john` → Output: `john:x:1000:1000:John Doe:/home/john:/bin/bash`.
  - Fields: `username:password:UID:GID:comment:home_dir:shell`.
  - Practical Insight: Use to check user settings (password field `x` means password is in `/etc/shadow`).

- **`cat /etc/shadow`**:
  - Purpose: Shows encrypted passwords and password policies (requires root).
  - Example: `sudo cat /etc/shadow | grep john` → Output: `john:$6$...:18600:0:99999:7:::`.
  - Fields: `username:password:last_changed:min_days:max_days:warning:inactive:expire:reserved`.
  - Practical Insight: Use for auditing password policies.

- **`cat /etc/group`**:
  - Purpose: Lists groups and their members.
  - Example: `cat /etc/group | grep sudo` → Output: `sudo:x:27:john,alice`.
  - Fields: `group_name:password:GID:member_list`.
  - Practical Insight: Check which users belong to a group.

- **`getent`**:
  - Purpose: Queries user or group databases (e.g., `/etc/passwd`, LDAP).
  - Examples:
    - `getent passwd john` → Show user details.
    - `getent group sudo` → Show group members.
  - Practical Insight: More reliable than `cat` for networked systems.

### 2. **Creating and Managing Users**

- **`useradd`**:
  - Purpose: Creates a new user (low-level, requires manual setup).
  - Options:
    - `-m`: Create home directory.
    - `-d /path`: Specify home directory.
    - `-s /path/shell`: Set default shell.
    - `-g group`: Set primary group.
    - `-G group1,group2`: Add to supplementary groups.
  - Example: `sudo useradd -m -s /bin/bash -g users -G sudo john` → Create user `john` with home and sudo access.
  - Practical Insight:
    - Sets up `/etc/passwd` and `/home/john` but doesn’t set a password (use `passwd`).
    - Common for scripting or minimal setups.

- **`adduser`**:
  - Purpose: Interactive, user-friendly alternative to `useradd`.
  - Example: `sudo adduser john` → Prompts for password, full name, and other details.
  - Practical Insight:
    - Automatically creates home directory, sets shell, and prompts for password.
    - Preferred for most admin tasks.

- **`passwd`**:
  - Purpose: Sets or changes a user’s password.
  - Examples:
    - `sudo passwd john` → Set password for `john`.
    - `passwd` → Change current user’s password.
  - Practical Insight:
    - Passwords are stored encrypted in `/etc/shadow`.
    - Use `passwd -l  to lock an account: `sudo passwd -l john`.

- **`usermod`**:
  - Purpose: Modifies an existing user account.
  - Options:
    - `-aG group`: Append user to supplementary groups.
    - `-s /path/shell`: Change shell.
    - `-d /path`: Change home directory.
    - `-l newname`: Rename user.
  - Examples:
    - `sudo usermod -aG developers john` → Add `john` to `developers` group.
    - `sudo usermod -s /bin/zsh john` → Change shell to Zsh.
  - Practical Insight: Use `-a` with `-G` to avoid overwriting existing groups.

- **`userdel`**:
  - Purpose: Deletes a user account.
  - Options:
    - `-r`: Remove home directory and mail spool.
  - Example: `sudo userdel -r john` → Delete `john` and their home directory.
  - Practical Insight:
    - Ensure user is logged out before deletion.
    - Check for owned files: `find / -user john` before deleting.

### 3. **Managing Groups**

- **`groupadd`**:
  - Purpose: Creates a new group.
  - Example: `sudo groupadd developers` → Create `developers` group.
  - Practical Insight: Use for shared access to files/directories.

- **`groupmod`**:
  - Purpose: Modifies a group.
  - Options:
    - `-n newname`: Rename group.
  - Example: `sudo groupmod -n devteam developers` → Rename group to `devteam`.
  - Practical Insight: Rarely used but handy for group reorganization.

- **`groupdel`**:
  - Purpose: Deletes a group.
  - Example: `sudo groupdel developers` → Delete `developers` group.
  - Practical Insight: Ensure no files are owned by the group: `find / -group developers`.

- **`gpasswd`**:
  - Purpose: Manages group membership and passwords.
  - Examples:
    - `sudo gpasswd -a john developers` → Add `john` to `developers`.
    - `sudo gpasswd -d john developers` → Remove `john` from `developers`.
  - Practical Insight: Alternative to `usermod -aG` for group membership.

### 4. **Sudo and Root Access**

- **Sudo**: Allows users to run commands as another user (usually root).
- **Configuration**: `/etc/sudoers` (edit with `visudo` for syntax checking).
- **Examples**:
  - `sudo apt update` → Run as root.
  - Add user to sudo group: `sudo usermod -aG sudo john`.
- **Sudoers Example**:
  ```bash
  john ALL=(ALL:ALL) ALL  # Allow john to run all commands as any user
  john ALL=(ALL) NOPASSWD: /usr/bin/apt  # Passwordless apt for john
  ```
- **Practical Insight**:
  - Use `sudo -i` for a root shell.
  - Audit `/etc/sudoers` for security.
  - Avoid direct root login; use `sudo` instead.

### 5. **Locking and Unlocking Accounts**

- **`passwd -l <user>`**:
  - Purpose: Locks a user account (disables login).
  - Example: `sudo passwd -l john` → Lock `john`’s account.
  - Practical Insight: Adds `!` to password in `/etc/shadow`.

- **`passwd -u <user>`**:
  - Purpose: Unlocks a user account.
  - Example: `sudo passwd -u john` → Unlock `john`’s account.
  - Practical Insight: Ensure password is set after unlocking.

- **`chage`**:
  - Purpose: Manages password aging and account expiration.
  - Examples:
    - `sudo chage -E 2025-12-31 john` → Set account expiration date.
    - `sudo chage -l john` → Show password policy details.
  - Practical Insight: Useful for temporary accounts or compliance.

## Practical Examples

1. **Create a New User**:
   - `sudo adduser alice` → Create user `alice` with home directory and password.
   - `sudo usermod -aG sudo alice` → Grant sudo privileges.
   - Verify: `id alice`.

2. **Set Up a Service Account**:
   - `sudo useradd -r -s /sbin/nologin myservice` → Create system user `myservice` (no login).
   - `sudo chown myservice:myservice /var/lib/myservice` → Assign directory ownership.
   - Verify: `cat /etc/passwd | grep myservice`.

3. **Create a Shared Group**:
   - `sudo groupadd projects`.
   - `sudo usermod -aG projects john`.
   - `sudo usermod -aG projects alice`.
   - `sudo mkdir /shared && sudo chgrp -R projects /shared && sudo chmod -R 2770 /shared`.
   - Verify: `ls -ld /shared`.

4. **Lock a User Account**:
   - `sudo passwd -l bob` → Lock `bob`’s account.
   - Test: `su - bob` (should fail).
   - Unlock: `sudo passwd -u bob`.

5. **Audit Users**:
   - List all users: `cat /etc/passwd`.
   - Check sudo users: `getent group sudo`.
   - Find files owned by a user: `find / -user john`.

## Troubleshooting User Issues

1. **Cannot Log In**:
   - Check `/etc/passwd` and `/etc/shadow` for correct entries.
   - Verify shell: `grep user /etc/passwd` (e.g., `/bin/bash`, not `/sbin/nologin`).
   - Unlock account: `sudo passwd -u user`.
   - Check password: `sudo passwd user`.

2. **Permission Denied**:
   - Verify group membership: `id user`.
   - Check file permissions: `ls -l file`.
   - Add to group: `sudo usermod -aG group user`.

3. **Sudo Not Working**:
   - Check group: `groups user` (ensure `sudo` or equivalent).
   - Inspect `/etc/sudoers`: `sudo visudo`.
   - Test: `sudo -l` (lists allowed commands).

4. **Duplicate UID/GID**:
   - Check for conflicts: `cat /etc/passwd | awk -F: '{print $3}' | sort | uniq -d`.
   - Fix: `sudo usermod -u <new_uid> user`.

5. **Home Directory Missing**:
   - Create: `sudo mkdir /home/user && sudo chown user:user /home/user`.
   - Update: `sudo usermod -d /home/user user`.

## Best Practices

1. **Use `adduser` Over `useradd`**:
   - More user-friendly and sets up defaults automatically.

2. **Restrict Root Access**:
   - Disable direct root login: `sudo passwd -l root`.
   - Use `sudo` for admin tasks.

3. **Secure Passwords**:
   - Enforce strong passwords via `/etc/security/pwquality.conf` (on some distros).
   - Set password aging: `sudo chage -M 90 user` (max 90 days).

4. **Use Groups for Collaboration**:
   - Create groups for shared resources (e.g., `developers` for `/shared`).
   - Use `setgid` on directories: `chmod 2770 /shared`.

5. **Audit Regularly**:
   - Check for unused accounts: `cat /etc/passwd`.
   - Remove unnecessary users: `sudo userdel -r user`.
   - Monitor sudo access: `getent group sudo`.

6. **Service Accounts**:
   - Use system users (`-r`) with no login shell (`/sbin/nologin`).
   - Restrict permissions: `chmod 700 /var/lib/service`.

## Practice Exercises

1. **Create and Configure a User**:
   - Create user `bob`: `sudo adduser bob`.
   - Add to `sudo` group: `sudo usermod -aG sudo bob`.
   - Verify: `id bob` and `sudo -l -U bob`.

2. **Set Up a Service Account**:
   - Create: `sudo useradd -r -s /sbin/nologin backup`.
   - Create directory: `sudo mkdir /backup && sudo chown backup:backup /backup`.
   - Verify: `cat /etc/passwd | grep backup`.

3. **Manage a Shared Group**:
   - Create group: `sudo groupadd team`.
   - Add users: `sudo gpasswd -a alice team && sudo gpasswd -a bob team`.
   - Set up directory: `sudo mkdir /team && sudo chgrp -R team /team && sudo chmod -R 2770 /team`.
   - Test: Log in as `alice`, create a file in `/team`, and check group: `ls -l /team`.

4. **Lock and Audit**:
   - Lock user: `sudo passwd -l tempuser`.
   - Check: `sudo cat /etc/shadow | grep tempuser`.
   - List all users: `getent passwd | cut -d: -f1`.

5. **Fix a Permission Issue**:
   - Create file: `sudo touch /data/file.txt && sudo chmod 600 /data/file.txt && sudo chown root:root /data/file.txt`.
   - Try accessing as `bob`: `cat /data/file.txt` (should fail).
   - Fix: `sudo chown bob:bob /data/file.txt && sudo chmod 644 /data/file.txt`.

## Cheat Sheet

| Task                     | Command Example                              |
|--------------------------|----------------------------------------------|
| Show current user        | `whoami`                                     |
| Show user details        | `id john`                                    |
| List users               | `cat /etc/passwd`                            |
| List groups              | `cat /etc/group`                             |
| Create user (interactive)| `sudo adduser john`                          |
| Create user (manual)     | `sudo useradd -m -s /bin/bash john`          |
| Set password             | `sudo passwd john`                           |
| Modify user              | `sudo usermod -aG sudo john`                 |
| Delete user              | `sudo userdel -r john`                       |
| Create group             | `sudo groupadd developers`                   |
| Add user to group        | `sudo gpasswd -a john developers`            |
| Remove user from group   | `sudo gpasswd -d john developers`            |
| Delete group             | `sudo groupdel developers`                   |
| Lock account             | `sudo passwd -l john`                        |
| Unlock account           | `sudo passwd -u john`                        |
| Set account expiration   | `sudo chage -E 2025-12-31 john`             |
| Edit sudoers             | `sudo visudo`                                |

## Resources

- **Man Pages**: `man useradd`, `man passwd`, `man sudoers`, `man chage`.
- **Online**: Linux User Management (e.g., Linux Journey, Arch Wiki, Ubuntu Server Guide).
- **Practice**: Use a VM to create users, groups, and test permissions.


---
