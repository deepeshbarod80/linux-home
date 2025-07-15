
# Linux File Permissions

Linux is a multi-user operating system, and file permissions determine **who** can **read**, **write**, or **execute** files and directories. Permissions are a cornerstone of Linux security, ensuring that users, applications, and services access only what they’re authorized to. Mastering permissions is essential for configuring servers, protecting sensitive data, and troubleshooting access issues.

## Key Concepts

- **Users and Groups**:
  - **User**: An individual account (e.g., `john`, `root`).
  - **Group**: A collection of users (e.g., `sudo`, `www-data`).
  - Every file is owned by a **user** and a **group**.
- **Permission Types**:
  - **Read (r)**: View file contents or list directory contents.
  - **Write (w)**: Modify file contents or create/delete files in a directory.
  - **Execute (x)**: Run a file (e.g., script, binary) or access a directory’s contents.
- **Permission Classes**:
  - **Owner (u)**: The user who owns the file.
  - **Group (g)**: The group associated with the file.
  - **Others (o)**: Everyone else.
- **File Types**:
  - Regular files (e.g., `file.txt`).
  - Directories (require `x` to enter).
  - Symbolic links, device files, etc.
- **Permission Representation**:
  - **Symbolic**: `rwxr-xr-x` (owner: rwx, group: r-x, others: r-x).
  - **Numeric (Octal)**: `755` (owner: 7, group: 5, others: 5).

## Viewing Permissions

- **Command**: `ls -l`
- **Output Example**:
  ```bash
  -rw-r--r-- 1 user group 123 Oct 10 12:00 file.txt
  drwxr-xr-x 2 user group 4096 Oct 10 12:00 mydir
  lrwxrwxrwx 1 user group   10 Oct 10 12:00 link -> file.txt
  ```
- **Breakdown**:
  - **First character**: File type (`-` for file, `d` for directory, `l` for symlink).
  - **Next 9 characters**: Permissions (3 sets of `rwx` for owner, group, others).
  - **Number**: Number of hard links.
  - **user group**: Owner and group.
  - **Size**: File size in bytes.
  - **Date/Time**: Last modified.
  - **Name**: File or directory name.

## Permission Details

### Symbolic Notation
- Format: `rwxrwxrwx` (owner, group, others).
- Each set of 3 characters represents permissions:
  - `r` = read (4 in octal).
  - `w` = write (2 in octal).
  - `x` = execute (1 in octal).
  - `-` = no permission (0 in octal).
- Example: `rw-r--r--`
  - Owner: `rw-` (read, write).
  - Group: `r--` (read only).
  - Others: `r--` (read only).

### Numeric (Octal) Notation
- Each permission class is represented by a number (0–7).
- Calculate by adding:
  - Read = 4.
  - Write = 2.
  - Execute = 1.
- Example: `755` = `rwxr-xr-x`
  - Owner: 7 (4+2+1 = rwx).
  - Group: 5 (4+1 = r-x).
  - Others: 5 (4+1 = r-x).
- Common Permissions:
  - `644`: Files (owner: rw, group/others: r).
  - `755`: Executables/directories (owner: rwx, group/others: rx).
  - `600`: Sensitive files (owner: rw, group/others: none).

### Directory Permissions
- **Read (r)**: List contents (`ls`).
- **Write (w)**: Create, delete, or rename files in the directory.
- **Execute (x)**: Enter the directory (`cd`) or access its files.
- Example:
  - `drwxr-xr-x` (755): Owner can modify, others can list and enter.
  - `drwx------` (700): Only owner can access or modify.

## Managing Permissions

### 1. **`chmod` (Change Mode)**

- **Purpose**: Modifies file or directory permissions.
- **Syntax**:
  - Symbolic: `chmod [class][+/-][permissions] file`
  - Numeric: `chmod [octal] file`
- **Options**:
  - `-R`: Recursive (apply to directory and its contents).
  - `-v`: Verbose (show changes).
- **Examples**:
  - Symbolic:
    - `chmod u+x script.sh` → Add execute permission for owner.
    - `chmod g-w file.txt` → Remove write permission for group.
    - `chmod o+r /var/www/html` → Add read permission for others.
  - Numeric:
    - `chmod 644 file.txt` → Set owner: rw, group/others: r.
    - `chmod 755 /usr/local/bin/myscript` → Set executable permissions.
  - Recursive:
    - `chmod -R 755 /var/www` → Apply to web directory and contents.
- **Practical Insight**:
  - Use numeric for absolute permissions (e.g., `644`).
  - Use symbolic for incremental changes (e.g., `u+x`).
  - Be cautious with `-R` to avoid over-permissive settings.

### 2. **`chown` (Change Owner)**

- **Purpose**: Changes the user and/or group ownership of a file or directory.
- **Syntax**: `chown [user][:group] file`
- **Options**:
  - `-R`: Recursive.
  - `-v`: Verbose.
- **Examples**:
  - `sudo chown user file.txt` → Change owner to `user`.
  - `sudo chown user:group file.txt` → Change owner and group.
  - `sudo chown -R www-data:www-data /var/www/html` → Set web server ownership.
  - `sudo chown :group file.txt` → Change group only.
- **Practical Insight**:
  - Common for services (e.g., `www-data` for web servers, `mysql` for databases).
  - Verify ownership: `ls -l`.

### 3. **`chgrp` (Change Group)**

- **Purpose**: Changes the group ownership of a file or directory.
- **Syntax**: `chgrp [group] file`
- **Options**:
  - `-R`: Recursive.
- **Example**: `sudo chgrp developers /home/project`
- **Practical Insight**: Less common than `chown`, which can handle both user and group.

## Special Permissions

Beyond `rwx`, Linux supports special permissions for advanced control:

1. **Setuid (Set User ID)**:
   - Purpose: Runs a file with the owner’s permissions, not the user’s.
   - Notation: `s` in owner’s execute position (e.g., `rwsr-xr-x`).
   - Octal: Add 4000 (e.g., `4755`).
   - Example: `/usr/bin/passwd` (allows users to change passwords as root).
   - Command: `chmod u+s file`.

2. **Setgid (Set Group ID)**:
   - Purpose: Runs a file with the group’s permissions or ensures new files in a directory inherit the directory’s group.
   - Notation: `s` in group’s execute position.
   - Octal: Add 2000.
   - Example: `chmod g+s /shared` → New files in `/shared` inherit the group.
   - Command: `chmod g+s dir`.

3. **Sticky Bit**:
   - Purpose: Prevents users from deleting files in a directory unless they own them (common for `/tmp`).
   - Notation: `t` in others’ execute position (e.g., `rwxrwxrwt`).
   - Octal: Add 1000.
   - Example: `chmod +t /tmp` → Only file owners can delete files in `/tmp`.
   - Command: `chmod +t dir`.

## Practical Examples

1. **Secure a Configuration File**:
   - Command: `sudo chmod 600 /etc/ssh/sshd_config`
   - Result: Owner (root) can read/write; no access for group/others.

2. **Make a Script Executable**:
   - Command: `chmod 755 myscript.sh`
   - Result: Owner can read/write/execute; others can read/execute.

3. **Set Up a Web Server Directory**:
   - Commands:
     ```bash
     sudo chown -R www-data:www-data /var/www/html
     sudo chmod -R 755 /var/www/html
     ```
   - Result: Web server (`www-data`) owns files; public can read but not modify.

4. **Shared Directory for a Team**:
   - Commands:
     ```bash
     sudo mkdir /shared
     sudo chgrp -R developers /shared
     sudo chmod -R 2775 /shared
     ```
   - Result: Group `developers` owns the directory; new files inherit the group (`setgid`); group members can read/write.

5. **Protect a Sensitive File**:
   - Command: `sudo chmod 400 /home/user/backup.key`
   - Result: Only owner can read; no write or execute.

## Common Permission Scenarios

- **Configuration Files** (`/etc`): Typically `644` (readable by all, writable by owner) or `600` (sensitive files like `/etc/shadow`).
- **Executables** (`/bin`, `/usr/bin`): Usually `755` (executable by all, writable by owner).
- **User Home Directories** (`/home/user`): Often `700` (private to owner) or `755` (readable by others).
- **Web Server Content** (`/var/www`): `755` for directories, `644` for files, owned by `www-data`.
- **Logs** (`/var/log`): Varies; often `640` (readable by admin group, writable by owner).

## Troubleshooting Permissions

1. **Permission Denied**:
   - Check permissions: `ls -l file`.
   - Verify user/group: `id` (shows your user and groups).
   - Adjust permissions: `sudo chmod` or `sudo chown`.
   - Example: Can’t access `/var/www/html`? Run `sudo chmod -R o+r /var/www/html`.

2. **Service Fails to Start**:
   - Check service logs: `journalctl -u nginx` or `cat /var/log/nginx/error.log`.
   - Verify file ownership (e.g., `www-data` for Nginx).
   - Example: `sudo chown nginx:nginx /var/log/nginx`.

3. **Cannot Delete Files**:
   - Check sticky bit on directories: `ls -ld /tmp`.
   - Ensure you own the file or use `sudo`.
   - Example: `sudo rm /tmp/lockedfile`.

4. **Script Won’t Run**:
   - Check execute permission: `ls -l script.sh`.
   - Add execute: `chmod +x script.sh`.
   - Ensure correct interpreter: `#!/bin/bash` at the top.

## Best Practices

1. **Principle of Least Privilege**:
   - Grant only the permissions needed (e.g., `600` for private keys, not `777`).
   - Avoid `777` (world-writable); it’s a security risk.

2. **Use Groups**:
   - Assign users to groups (e.g., `developers`) for shared access.
   - Use `setgid` for collaborative directories.

3. **Backup Before Changing**:
   - Copy critical files: `sudo cp /etc/file /etc/file.bak`.
   - Test changes in a non-production environment.

4. **Audit Permissions**:
   - Regularly check sensitive directories: `ls -l /etc`, `ls -ld /var/www`.
   - Use `find` to identify misconfigured files: `find / -perm -o+w` (world-writable files).

5. **Server-Specific Permissions**:
   - Web servers: `755` for dirs, `644` for files, owned by service user (e.g., `www-data`).
   - Databases: `700` for data dirs (e.g., `/var/lib/mysql`), owned by `mysql`.
   - SSH: `600` for `~/.ssh/id_rsa`, `644` for `~/.ssh/authorized_keys`.

## Practice Exercises

1. **Basic Permissions**:
   - Create a file: `touch /tmp/test.txt`.
   - Set permissions: `chmod 640 /tmp/test.txt`.
   - Verify: `ls -l /tmp/test.txt` (should show `-rw-r-----`).
   - Change owner: `sudo chown user:group /tmp/test.txt`.

2. **Executable Script**:
   - Create a script: `echo -e '#!/bin/bash\necho "Hello"' > myscript.sh`.
   - Make executable: `chmod +x myscript.sh`.
   - Run: `./myscript.sh`.
   - Restrict to owner: `chmod 700 myscript.sh`.

3. **Web Server Setup**:
   - Create a test web dir: `sudo mkdir -p /var/www/test`.
   - Set permissions: `sudo chmod -R 755 /var/www/test`.
   - Set ownership: `sudo chown -R www-data:www-data /var/www/test`.
   - Verify: `ls -ld /var/www/test`.

4. **Shared Directory**:
   - Create: `sudo mkdir /shared`.
   - Set group: `sudo chgrp developers /shared`.
   - Set permissions with setgid: `sudo chmod 2770 /shared`.
   - Test: Create a file as a group member and verify group ownership.

5. **Find Misconfigured Files**:
   - Search for world-writable files: `find /home -perm -o+w`.
   - Fix a file: `chmod o-w /home/user/public.txt`.

## Cheat Sheet

| Task                     | Command Example                              |
|--------------------------|----------------------------------------------|
| View permissions         | `ls -l file.txt`                             |
| Set permissions (numeric)| `chmod 644 file.txt`                         |
| Set permissions (symbolic)| `chmod u+x script.sh`                       |
| Recursive permissions     | `chmod -R 755 /var/www`                      |
| Change owner             | `chown user file.txt`                        |
| Change owner and group   | `chown user:group file.txt`                  |
| Recursive ownership      | `chown -R www-data:www-data /var/www`        |
| Change group             | `chgrp group file.txt`                       |
| Set setuid               | `chmod u+s file`                             |
| Set setgid               | `chmod g+s dir`                              |
| Set sticky bit           | `chmod +t dir`                               |

## Resources

- **Man Pages**: `man chmod`, `man chown`, `man ls`.
- **Online**: Linux Permissions Guide (e.g., Linux Journey, Arch Wiki).
- **Practice**: Experiment on a VM or use tools like `stat` for detailed file info.

---

### Next Steps
- Practice these commands on your Linux machine (e.g., set permissions for a test file, configure a web directory, or troubleshoot access issues).
- Share the next topic (e.g., “Process Management” or “System Services”) or ask for clarification on permissions, and I’ll provide clear, practical notes.
- Want a challenge? Try scenarios like “Secure an SSH key pair” or “Set up a shared directory for a team”.

---