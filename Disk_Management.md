
# Linux Disk Management

**Disk management** in Linux involves configuring, partitioning, formatting, mounting, and monitoring storage devices (e.g., hard drives, SSDs, USB drives) to ensure data availability, performance, and redundancy. It’s essential for setting up servers, managing backups, and optimizing storage for applications like databases or web servers.

## Key Concepts

- **Storage Devices**: Physical or virtual devices (e.g., `/dev/sda`, `/dev/nvme0n1`).
  - **Block Devices**: Handle data in fixed-size blocks (e.g., disks).
  - **Device Naming**:
    - SATA/SCSI: `/dev/sda`, `/dev/sdb` (e.g., `sda1` for first partition).
    - NVMe: `/dev/nvme0n1` (e.g., `nvme0n1p1` for first partition).
    - Virtual: `/dev/vda` (common in VMs).
- **Partitions**: Divisions of a disk (e.g., `/dev/sda1`) for organizing data.
- **File Systems**: Formats for storing data (e.g., ext4, xfs, btrfs, ntfs).
- **Mount Points**: Directories where file systems are accessed (e.g., `/mnt`, `/home`).
- **Logical Volume Manager (LVM)**: Flexible storage management with resizable volumes.
- **RAID**: Combines disks for redundancy or performance (e.g., RAID 0, 1, 5).
- **Practical Use**: Disk management is critical for provisioning storage, recovering data, expanding volumes, and monitoring disk health.

## Core Disk Management Components

1. **Partition Tools**:
   - `fdisk`, `parted`, `gparted`: Create and manage partitions.
   - `lsblk`, `blkid`: List devices and file systems.

2. **File System Tools**:
   - `mkfs`: Format partitions (e.g., `mkfs.ext4`).
   - `fsck`: Check and repair file systems.

3. **Mounting**:
   - `/etc/fstab`: Configures automatic mounts.
   - `mount`, `umount`: Manual mounting/unmounting.

4. **LVM**:
   - Physical Volumes (PV), Volume Groups (VG), Logical Volumes (LV).
   - Tools: `pvcreate`, `vgcreate`, `lvcreate`.

5. **Monitoring**:
   - `df`, `du`: Disk usage.
   - `iostat`, `smartctl`: Disk performance and health.

## Essential Disk Management Commands

Below are the core commands and tools for disk management, with explanations, options, and practical examples.

### 1. **Listing and Identifying Disks**

- **`lsblk`**:
  - Purpose: Lists block devices and their mount points in a tree view.
  - Options:
    - `-f`: Show file system types.
    - `-a`: Include empty devices.
  - Example: `lsblk -f` → Output:
    ```
    NAME        FSTYPE  MOUNTPOINT
    sda
    ├─sda1      ext4    /boot
    ├─sda2      ext4    /
    └─sda3      swap
    sdb
    └─sdb1      xfs     /mnt/data
    ```
  - **Practical Insight**: Use to identify available disks and partitions.

- **`blkid`**:
  - Purpose: Shows UUIDs and file system types of partitions.
  - Example: `sudo blkid` → Output:
    ```
    /dev/sda1: UUID="1234-5678" TYPE="ext4"
    /dev/sdb1: UUID="abcd-efgh" TYPE="xfs"
    ```
  - **Practical Insight**: Use UUIDs in `/etc/fstab` for reliable mounting.

- **`fdisk -l`**:
  - Purpose: Lists all disks and partitions (requires root).
  - Example: `sudo fdisk -l` → Shows disk sizes, partitions, and types.
  - **Practical Insight**: Use to inspect disk layout before partitioning.

- **`parted -l`**:
  - Purpose: Lists partition tables and disk details.
  - Example: `sudo parted -l` → Shows partition types (e.g., GPT, MBR).
  - **Practical Insight**: Supports larger disks (>2TB) compared to `fdisk`.

### 2. **Partitioning Disks**

- **`fdisk`**:
  - Purpose: Creates and manages partitions on MBR disks.
  - Example: Partition a new disk (`/dev/sdb`):
    ```bash
    sudo fdisk /dev/sdb
    # Commands:
    # n (new partition), p (primary), 1 (number), default (start), default (end)
    # t (type), 83 (Linux)
    # w (write and exit)
    ```
  - **Practical Insight**: Use for disks <2TB; run `partprobe` to refresh partition table.

- **`parted`**:
  - Purpose: Manages partitions on both MBR and GPT disks.
  - Example: Create a GPT partition:
    ```bash
    sudo parted /dev/sdb
    (parted) mklabel gpt
    (parted) mkpart primary ext4 0% 100%
    (parted) set 1 lvm on  # Optional: For LVM
    (parted) quit
    ```
  - **Practical Insight**: Ideal for large disks; supports interactive and scripted use.

- **`gparted`** (GUI):
  - Purpose: Graphical partition editor.
  - Install: `sudo apt install gparted`.
  - Example: Run `sudo gparted`, select `/dev/sdb`, create partitions visually.
  - **Practical Insight**: Beginner-friendly; use on non-production systems.

### 3. **Formatting File Systems**

- **`mkfs`**:
  - Purpose: Formats partitions with a file system.
  - Examples:
    - `sudo mkfs.ext4 /dev/sdb1` → Format as ext4.
    - `sudo mkfs.xfs /dev/sdb1` → Format as xfs.
    - `sudo mkfs.vfat /dev/sdb1` → Format as FAT32 (for USB drives).
  - **Practical Insight**:
    - Ext4: General-purpose, reliable.
    - Xfs: High-performance for large files.
    - Btrfs: Advanced features like snapshots.

- **`fsck`**:
  - Purpose: Checks and repairs file systems.
  - Example: `sudo fsck /dev/sdb1` → Check for errors.
  - Options:
    - `-y`: Auto-fix errors.
    - `-f`: Force check.
  - **Practical Insight**: Run on unmounted partitions; use before mounting.

### 4. **Mounting File Systems**

- **`mount`**:
  - Purpose: Attaches a file system to a directory.
  - Examples:
    - `sudo mount /dev/sdb1 /mnt` → Mount `sdb1` to `/mnt`.
    - `sudo mount -o ro /dev/sdb1 /mnt` → Mount read-only.
  - **Practical Insight**: Changes are temporary unless added to `/etc/fstab`.

- **`umount`**:
  - Purpose: Detaches a file system.
  - Example: `sudo umount /mnt` → Unmount `/mnt`.
  - **Practical Insight**: Ensure no processes are using the mount point (`lsof /mnt`).

- **`/etc/fstab`**:
  - Purpose: Configures automatic mounts at boot.
  - Example Entry:
    ```bash
    UUID=abcd-efgh /mnt/data xfs defaults 0 0
    ```
  - Fields: `device`, `mountpoint`, `fstype`, `options`, `dump`, `fsck`.
  - Test: `sudo mount -a` → Apply `fstab` without reboot.
  - **Practical Insight**:
    - Use UUIDs (`blkid`) for reliability.
    - Backup before editing: `sudo cp /etc/fstab /etc/fstab.bak`.

### 5. **Logical Volume Manager (LVM)**

- **Overview**: LVM provides flexible storage by abstracting physical disks into resizable logical volumes.
- **Steps to Set Up LVM**:
  1. **Create Physical Volume (PV)**:
     ```bash
     sudo pvcreate /dev/sdb1
     ```
  2. **Create Volume Group (VG)**:
     ```bash
     sudo vgcreate my_vg /dev/sdb1
     ```
  3. **Create Logical Volume (LV)**:
     ```bash
     sudo lvcreate -L 10G -n my_lv my_vg
     ```
  4. **Format and Mount**:
     ```bash
     sudo mkfs.ext4 /dev/my_vg/my_lv
     sudo mkdir /mnt/my_lv
     sudo mount /dev/my_vg/my_lv /mnt/my_lv
     ```
  - **Add to `/etc/fstab`**:
    ```bash
    /dev/my_vg/my_lv /mnt/my_lv ext4 defaults 0 0
    ```
- **Resize LV**:
  - Extend: `sudo lvextend -L +5G /dev/my_vg/my_lv; sudo resize2fs /dev/my_vg/my_lv`.
  - Reduce (careful): `sudo resize2fs /dev/my_vg/my_lv 8G; sudo lvreduce -L 8G /dev/my_vg/my_lv`.
- **Practical Insight**:
  - LVM allows dynamic resizing without downtime.
  - Use `pvs`, `vgs`, `lvs` to inspect PVs, VGs, and LVs.

### 6. **RAID (Redundant Array of Independent Disks)**

- **Overview**: Combines disks for redundancy or performance.
- **Common Levels**:
  - RAID 0: Striping (speed, no redundancy).
  - RAID 1: Mirroring (redundancy, no speed gain).
  - RAID 5: Striping + parity (balance, needs 3+ disks).
- **Tool**: `mdadm` (software RAID).
- **Example (RAID 1)**:
  ```bash
  sudo apt install mdadm
  sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
  sudo mkfs.ext4 /dev/md0
  sudo mkdir /mnt/raid
  sudo mount /dev/md0 /mnt/raid
  ```
- **Add to `/etc/fstab`**:
  ```bash
  /dev/md0 /mnt/raid ext4 defaults 0 0
  ```
- **Check Status**: `cat /proc/mdstat` or `sudo mdadm --detail /dev/md0`.
- **Practical Insight**:
  - Use for critical data (e.g., RAID 1 for `/home`).
  - Monitor disk failures: `sudo mdadm --monitor /dev/md0`.

### 7. **Monitoring Disk Usage and Health**

- **`df`**:
  - Purpose: Shows disk usage.
  - Example: `df -h` → Human-readable output:
    ```
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/sda2        50G   20G   30G  40% /
    /dev/sdb1       100G   10G   90G  10% /mnt/data
    ```
  - **Practical Insight**: Use to detect full disks.

- **`du`**:
  - Purpose: Estimates directory/file usage.
  - Example: `du -sh /var/log` → Show total size of `/var/log`.
  - **Practical Insight**: Use `du -ah / | sort -rh | head` to find large files.

- **`iostat`**:
  - Purpose: Monitors disk I/O (requires `sysstat`).
  - Example: `iostat -dx 1` → Show I/O stats every second.
  - **Practical Insight**: High `%util` indicates disk bottlenecks.

- **`smartctl`** (SMART Monitoring):
  - Purpose: Checks disk health (requires `smartmontools`).
  - Install: `sudo apt install smartmontools`.
  - Example: `sudo smartctl -a /dev/sda` → Show disk health, errors, and wear.
  - **Practical Insight**: Enable monitoring: `sudo systemctl enable smartd`.

## Practical Examples

1. **Add and Mount a New Disk**:
   - List disks: `lsblk`.
   - Partition: `sudo fdisk /dev/sdb` (create `sdb1`).
   - Format: `sudo mkfs.ext4 /dev/sdb1`.
   - Mount: `sudo mkdir /mnt/data; sudo mount /dev/sdb1 /mnt/data`.
   - Add to `fstab`:
     ```bash
     sudo blkid /dev/sdb1  # Get UUID
     sudo nano /etc/fstab
     # Add: UUID=abcd-efgh /mnt/data ext4 defaults 0 0
     sudo mount -a
     ```

2. **Set Up LVM**:
   - Prepare disk: `sudo pvcreate /dev/sdb1`.
   - Create VG: `sudo vgcreate data_vg /dev/sdb1`.
   - Create LV: `sudo lvcreate -L 20G -n data_lv data_vg`.
   - Format and mount: `sudo mkfs.xfs /dev/data_vg/data_lv; sudo mount /dev/data_vg/data_lv /mnt/data`.
   - Verify: `lvs; df -h /mnt/data`.

3. **Create a RAID 1 Array**:
   - Install: `sudo apt install mdadm`.
   - Create: `sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc`.
   - Format: `sudo mkfs.ext4 /dev/md0`.
   - Mount: `sudo mkdir /mnt/raid; sudo mount /dev/md0 /mnt/raid`.
   - Save config: `sudo mdadm --detail --scan >> /etc/mdadm/mdadm.conf`.
   - Verify: `cat /proc/mdstat`.

4. **Check Disk Health**:
   - Install: `sudo apt install smartmontools`.
   - Check: `sudo smartctl -a /dev/sda`.
   - Enable monitoring: `sudo systemctl enable smartd && sudo systemctl start smartd`.

5. **Troubleshoot Disk Space**:
   - Check usage: `df -h`.
   - Find large files: `sudo du -ah / | sort -rh | head -n 10`.
   - Clear logs: `sudo find /var/log -type f -name "*.gz" -delete`.

## Troubleshooting Disk Issues

1. **Disk Not Detected**:
   - Check: `lsblk` or `dmesg | grep disk`.
   - Rescan: `sudo echo "- - -" > /sys/class/scsi_host/hostX/scan` (replace `X` with host number).
   - Verify hardware: `sudo fdisk -l`.

2. **Mount Fails**:
   - Check `fstab`: `sudo mount -a` (look for errors).
   - Verify file system: `sudo fsck /dev/sdb1`.
   - Check mount point: `lsof /mnt/data` (ensure it’s free).

3. **Disk Full**:
   - Identify: `df -h`.
   - Find culprits: `du -sh /var/* | sort -rh`.
   - Clear space: `sudo truncate -s 0 /var/log/large.log`.

4. **LVM Issues**:
   - Check: `pvs; vgs; lvs`.
   - Resize errors: Ensure file system supports resizing (`resize2fs` for ext4, `xfs_growfs` for xfs).
   - Recover VG: `vgcfgrestore -f /etc/lvm/backup/my_vg my_vg`.

5. **RAID Degraded**:
   - Check: `cat /proc/mdstat` (look for `[U_]` or `[_U]`).
   - Replace disk: `sudo mdadm /dev/md0 --add /dev/sdd --replace /dev/sdb`.
   - Monitor rebuild: `watch cat /proc/mdstat`.

## Best Practices

1. **Plan Partitioning**:
   - Separate `/`, `/home`, `/var`, `/boot` for flexibility.
   - Use LVM for dynamic resizing.

2. **Use Reliable File Systems**:
   - Ext4: Stable, general-purpose.
   - Xfs: High-performance for large files.
   - Btrfs: For snapshots and checksumming (advanced).

3. **Automate Mounts**:
   - Use `/etc/fstab` with UUIDs.
   - Test: `sudo mount -a` before reboot.

4. **Monitor Disk Health**:
   - Enable SMART: `sudo systemctl enable smartd`.
   - Check regularly: `sudo smartctl -a /dev/sda`.
   - Set up alerts: Use Prometheus/Grafana (from monitoring topic).

5. **Backup Before Changes**:
   - Save `fstab`: `sudo cp /etc/fstab /etc/fstab.bak`.
   - Back up data: `rsync -av /mnt/data /backup`.

6. **Secure Disks**:
   - Encrypt sensitive partitions: Use `cryptsetup` (`sudo cryptsetup luksFormat /dev/sdb1`).
   - Restrict access: `chmod 700 /mnt/data; chown user:group /mnt/data`.

## Practice Exercises

1. **Explore Disks**:
   - List devices: `lsblk -f`.
   - Check UUIDs: `sudo blkid`.
   - View partitions: `sudo fdisk -l`.

2. **Partition and Mount a Disk**:
   - Partition: `sudo fdisk /dev/sdb` (create one primary partition).
   - Format: `sudo mkfs.ext4 /dev/sdb1`.
   - Mount: `sudo mkdir /mnt/newdisk; sudo mount /dev/sdb1 /mnt/newdisk`.
   - Add to `fstab`: Use UUID, test with `sudo mount -a`.

3. **Set Up LVM**:
   - Create PV, VG, LV: Follow steps above for `/dev/sdb1`.
   - Mount: `sudo mount /dev/my_vg/my_lv /mnt/my_lv`.
   - Resize: Extend LV by 5GB and verify with `df -h`.

4. **Create a RAID Array**:
   - Simulate RAID 1: Use two loop devices (`sudo losetup /dev/loop0 disk1.img`).
   - Create: `sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/loop0 /dev/loop1`.
   - Mount and test: Format, mount, and check `/proc/mdstat`.

5. **Monitor Disk Usage**:
   - Check: `df -h; du -sh /var/log`.
   - Install SMART: `sudo apt install smartmontools`.
   - Run: `sudo smartctl -a /dev/sda`.

## Cheat Sheet

| Task                          | Command Example                                      |
|-------------------------------|-----------------------------------------------------|
| List disks                    | `lsblk -f`                                          |
| Show UUIDs                    | `sudo blkid`                                        |
| Partition disk                | `sudo fdisk /dev/sdb`                               |
| Create GPT partition          | `sudo parted /dev/sdb mkpart primary ext4 0% 100%` |
| Format partition              | `sudo mkfs.ext4 /dev/sdb1`                          |
| Check file system             | `sudo fsck /dev/sdb1`                               |
| Mount file system             | `sudo mount /dev/sdb1 /mnt`                         |
| Unmount                       | `sudo umount /mnt`                                  |
| Edit fstab                    | `sudo nano /etc/fstab`                              |
| Create LVM PV                 | `sudo pvcreate /dev/sdb1`                           |
| Create LVM VG                 | `sudo vgcreate my_vg /dev/sdb1`                     |
| Create LVM LV                 | `sudo lvcreate -L 10G -n my_lv my_vg`               |
| Resize LV                     | `sudo lvextend -L +5G /dev/my_vg/my_lv`             |
| Create RAID 1                 | `sudo mdadm --create /dev/md0 --level=1 --raid-devices=2` |
| Check RAID status             | `cat /proc/mdstat`                                  |
| Check disk usage              | `df -h`                                             |
| Find large files              | `du -ah / | sort -rh | head`                        |
| Monitor I/O                   | `iostat -dx 1`                                      |
| Check disk health             | `sudo smartctl -a /dev/sda`                         |

## Resources

- **Man Pages**: `man fdisk`, `man mkfs`, `man mount`, `man lvm`, `man mdadm`.
- **Online**:
  - Linux Disk Management: Arch Wiki, Linux Journey, Red Hat Storage Guide.
  - Tools: LVM (`lvm.org`), mdadm (`raid.wiki.kernel.org`), SMART (`smartmontools.org`).
- **Practice**: Use a VM with additional virtual disks to practice partitioning, LVM, and RAID.

---

### Next Steps
- Practice disk management on a Linux VM (e.g., partition a disk, set up LVM, create a RAID array).
- Share the next topic (e.g., “Backup and Recovery,” “Containerization,” or “Automation”) or ask for clarification, and I’ll provide clear, practical notes.
- Want a challenge? Try scenarios like “Expand an LVM volume for a growing database” or “Recover a corrupted file system.”

---
