# USB Pendrive Access Denied - Troubleshooting Guide (Linux)

## Problem
Unable to access USB pendrive on Linux PC - getting "Access Denied" or permission errors.

---

## Solution 1: Check Current User Permissions

### Step 1: Identify the USB device
```bash
lsblk
# or
sudo fdisk -l
```
Look for your pendrive (usually `/dev/sdb1`, `/dev/sdc1`, etc.)

### Step 2: Check current permissions
```bash
ls -l /dev/sdb1
# Replace sdb1 with your actual device
```

### Step 3: Check if device is mounted
```bash
mount | grep sdb1
# or
df -h
```

---

## Solution 2: Mount with Proper Permissions

### If not mounted, create mount point and mount:
```bash
# Create mount point
sudo mkdir -p /mnt/usb

# Mount with read-write permissions
sudo mount -o uid=$(id -u),gid=$(id -g) /dev/sdb1 /mnt/usb

# Now access your files
cd /mnt/usb
ls -la
```

### If already mounted, remount with proper permissions:
```bash
# Unmount first
sudo umount /dev/sdb1

# Remount with your user permissions
sudo mount -o uid=$(id -u),gid=$(id -g) /dev/sdb1 /mnt/usb
```

---

## Solution 3: Add Your User to Required Groups

```bash
# Add user to plugdev group (for USB device access)
sudo usermod -a -G plugdev $USER

# Add user to disk group (use with caution)
sudo usermod -a -G disk $USER

# Log out and log back in for changes to take effect
```

---

## Solution 4: Change Ownership/Permissions

### For mounted filesystem:
```bash
# Change ownership to your user
sudo chown -R $USER:$USER /mnt/usb

# Give full permissions
sudo chmod -R 755 /mnt/usb
```

### For the device itself:
```bash
# Change device permissions (temporary - resets on reboot)
sudo chmod 666 /dev/sdb1
```

---

## Solution 5: Check Filesystem Type and Repair

### Check filesystem type:
```bash
sudo blkid /dev/sdb1
```

### For FAT32/VFAT filesystems:
```bash
# Unmount first
sudo umount /dev/sdb1

# Check and repair
sudo fsck.vfat -a /dev/sdb1

# Remount
sudo mount -t vfat -o uid=$(id -u),gid=$(id -g),umask=0022 /dev/sdb1 /mnt/usb
```

### For NTFS filesystems:
```bash
# Install ntfs-3g if not present (you mentioned no internet, skip if not installed)
# sudo apt-get install ntfs-3g

# Unmount
sudo umount /dev/sdb1

# Check and repair
sudo ntfsfix /dev/sdb1

# Mount with ntfs-3g
sudo mount -t ntfs-3g -o uid=$(id -u),gid=$(id -g),permissions /dev/sdb1 /mnt/usb
```

### For ext4/ext3 filesystems:
```bash
# Unmount
sudo umount /dev/sdb1

# Check and repair
sudo e2fsck -f /dev/sdb1

# Mount
sudo mount /dev/sdb1 /mnt/usb
sudo chown -R $USER:$USER /mnt/usb
```

---

## Solution 6: Disable Write Protection (if hardware switch exists)

1. **Check physical write-protect switch** on the USB drive
2. **Check if drive is read-only:**
```bash
sudo hdparm -r /dev/sdb
```

3. **Try to disable read-only mode:**
```bash
sudo hdparm -r0 /dev/sdb
```

---

## Solution 7: Use Root Access (Temporary Solution)

```bash
# Open file manager as root (use carefully)
sudo nautilus /mnt/usb
# or
sudo thunar /mnt/usb
# or
sudo pcmanfm /mnt/usb

# Or copy files using sudo
sudo cp /mnt/usb/file.txt ~/Desktop/
sudo chown $USER:$USER ~/Desktop/file.txt
```

---

## Solution 8: Check SELinux/AppArmor Restrictions

### For SELinux:
```bash
# Check if SELinux is enforcing
getenforce

# Temporarily set to permissive (resets on reboot)
sudo setenforce 0

# Try accessing pendrive again
```

### For AppArmor:
```bash
# Check AppArmor status
sudo aa-status

# Temporarily disable (if needed)
sudo systemctl stop apparmor
```

---

## Solution 9: Auto-mount Configuration

### Create permanent mount entry in /etc/fstab:
```bash
# First, get UUID of your device
sudo blkid /dev/sdb1

# Edit fstab
sudo nano /etc/fstab

# Add this line (replace UUID and filesystem type):
UUID=YOUR-UUID-HERE /mnt/usb vfat uid=1000,gid=1000,umask=0022 0 0

# Save and test
sudo mount -a
```

---

## Solution 10: Check Disk Errors and Bad Sectors

```bash
# Check for bad sectors
sudo badblocks -v /dev/sdb1

# Check SMART status (if supported)
sudo smartctl -a /dev/sdb
```

---

## Quick Command Reference

### Essential Commands:
```bash
# 1. Identify device
lsblk

# 2. Create mount point
sudo mkdir -p /mnt/usb

# 3. Mount with your user permissions
sudo mount -o uid=$(id -u),gid=$(id -g) /dev/sdb1 /mnt/usb

# 4. Access files
cd /mnt/usb

# 5. When done, safely unmount
sudo umount /mnt/usb
```

---

## Common Error Messages and Solutions

| Error Message | Solution |
|---------------|----------|
| "Permission denied" | Use Solution 2 or 4 |
| "Read-only file system" | Use Solution 5 or 6 |
| "Device is busy" | Close all programs using the drive, then unmount |
| "Mount point does not exist" | Create mount point: `sudo mkdir -p /mnt/usb` |
| "Unknown filesystem type" | Check filesystem with `blkid` and install appropriate tools |
| "Structure needs cleaning" | Use `fsck` or `ntfsfix` (Solution 5) |

---

## Important Notes

1. **Replace `/dev/sdb1`** with your actual device name (check with `lsblk`)
2. **Always unmount before removing:** `sudo umount /mnt/usb`
3. **Backup important data** before running repair commands
4. **Be careful with sudo commands** - they have system-wide effects
5. **Most permission changes are temporary** and reset on reboot for security

---

## If Nothing Works

1. Try the pendrive on another Linux machine to rule out hardware failure
2. Check if the pendrive works on Windows (if dual-boot available)
3. The pendrive might be physically damaged or have corrupted firmware
4. Consider data recovery tools if data is critical

---

## No Internet Workaround

Since you don't have internet access:
- Most solutions above use built-in Linux commands
- If you need additional tools (ntfs-3g, etc.), you'll need to:
  - Download packages on another computer with internet
  - Transfer via another USB drive
  - Install manually using `dpkg -i package.deb` (Debian/Ubuntu) or `rpm -i package.rpm` (Fedora/RHEL)

