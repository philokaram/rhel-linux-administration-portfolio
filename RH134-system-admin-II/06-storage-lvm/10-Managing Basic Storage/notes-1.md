# Creating and Managing File Systems on Standard Partitions — Notes

## 1. What Is Partitioning?

- **Partitioning** = dividing a block device into more manageable units
- Use partitions when you need **separate file systems** for different use cases
- If you want to use the **entire block device** for one file system → partitions are **not required**

> 💡 **Partitions are overrated** unless you specifically need to divide the space.

---

## 2. Partitioning Schemes — MS-DOS (MBR) vs GPT

| Feature | MS-DOS (MBR) | GPT (GUID Partition Table) |
|---------|--------------|----------------------------|
| Max partitions | 4 primary (or 3 primary + 1 extended) | 128 primary partitions |
| Extended/logical partitions | ✅ Yes (workaround for >4) | ❌ No (not needed) |
| Boot from >2TB disk | ❌ No | ✅ Yes |
| Partition numbering | 1-4 primary, 5+ logical | 1-128 primary |
| Age | Old (legacy) | Modern (recommended) |

### MS-DOS numbering:

```
/dev/sda1  ← Primary 1
/dev/sda2  ← Primary 2
/dev/sda3  ← Primary 3
/dev/sda4  ← Extended (always #5 if extended partition exists)
/dev/sda5  ← Extended partition itself
/dev/sda6  ← Logical partition 1
/dev/sda7  ← Logical partition 2
```

> 💡 Extended partitions always get number 5 (1-4 reserved for primary partitions).

---

## 3. Partitioning Tools — Comparison

| Tool | Interactive | Non-Interactive | Complexity | Recommendation |
|------|-------------|-----------------|------------|----------------|
| **cfdisk** | ✅ Yes (NCurses) | ❌ No | Low | **Recommended** |
| **fdisk** | ✅ Yes | ❌ No | Medium | Good alternative |
| **parted** | ✅ Yes | ✅ Yes | High | Not recommended (complex) |
| **Cockpit** | ✅ Yes (GUI) | ❌ No | Very Low | **Highly recommended** |

> 💡 **Work smarter:** Use `cfdisk` or Cockpit for partitioning. Avoid `parted` unless forced.

### Install cfdisk/fdisk:

```bash
dnf install -y util-linux
```

### Find which package provides a tool:

```bash
dnf provides */cfdisk
```

---

## 4. Viewing Block Devices — `lsblk -fp`

### Before any partitioning:

```bash
lsblk -fp
```

| Option | Meaning |
|--------|---------|
| `-f` | Show file system information |
| `-p` | Show full device paths |

**Shows:**
- Block devices
- Partitions (if any)
- File systems and UUIDs
- Mount points

---

## 5. Creating Partitions with `cfdisk` (Interactive, NCurses)

### Step 1 — Start cfdisk:

```bash
cfdisk /dev/sdd
```

### Step 2 — Choose label type:

- `gpt` (recommended for modern systems)
- `dos` (legacy MBR)

### Step 3 — Create partitions:

1. Highlight **Free space**
2. Select **New**
3. Enter size (e.g., `2G`, `1G`)
4. Press **Enter**

### Step 4 — Write changes:

1. Select **Write**
2. Type `yes` to confirm
3. Select **Quit**

### Verify:

```bash
lsblk -fp
```

---

## 6. Creating File Systems — `mkfs`

### XFS (default on RHEL):

```bash
mkfs.xfs /dev/sdd1
```

or

```bash
mkfs -t xfs /dev/sdd1
```

### EXT4 (alternative):

```bash
mkfs.ext4 /dev/sdd2
```

> ⚠️ **Warning:** `mkfs` will warn you if a file system already exists on the device.

---

## 7. Mounting File Systems — Temporary vs Persistent

### Temporary (lost on reboot):

```bash
mkdir -p /mnt/data
mount /dev/sdd1 /mnt/data
```

### Persistent — `/etc/fstab`

**Format:**
```
UUID=<uuid> <mountpoint> <fstype> <options> <dump> <fsck>
```

| Field | Purpose | Example |
|-------|---------|---------|
| 1 | UUID of the file system | `UUID=abc123...` |
| 2 | Mount point | `/mnt/data` |
| 3 | File system type | `xfs` |
| 4 | Mount options | `defaults` |
| 5 | Dump backup (0=no) | `0` |
| 6 | fsck order (0=no, 1=root, 2=other) | `0` |

### Example `/etc/fstab` entry:

```
UUID=12345678-1234-1234-1234-123456789abc /mnt/data xfs defaults 0 0
```

### Where to get the UUID:

```bash
lsblk -fp /dev/sdd1
```

or

```bash
blkid /dev/sdd1
```

> 💡 **Always use UUID** instead of device file names (`/dev/sdd1`) — device names can change.

---

## 8. Testing `/etc/fstab` — Critical Exam Tip

### Step 1 — Unmount the file system (if currently mounted):

```bash
umount /mnt/data
```

### Step 2 — Process fstab verbosely:

```bash
mount -av
```

| Option | Meaning |
|--------|---------|
| `-a` | Mount all file systems in `/etc/fstab` |
| `-v` | Verbose — show what's happening |

### Step 3 — Verify:

```bash
lsblk -fp
```

> ⚠️ **Exam tip:** If `/etc/fstab` has a syntax error, the system may boot into **maintenance mode**, and you won't get points for other work on that system. Always test `mount -av` before rebooting.

---

## 9. Using Cockpit (Web Console) — The Easiest Way

### Install Cockpit storage plugin:

```bash
dnf install -y cockpit-storaged
```

### Steps:

1. Connect to `https://server:9090`
2. Log in (e.g., `student`)
3. Navigate to **Storage** → **Disks**
4. Select the disk
5. Click **Create partition** (or format an existing one)
6. Set:
   - Size
   - File system type
   - Label
   - Mount point
7. Click **Format and mount**

> 💡 Cockpit creates **persistent** mounts automatically (updates `/etc/fstab`).

---

## 10. `udevadm settle` — Re-reading Partition Table

### Use when partitioning a device that is **currently in use**:

```bash
udevadm settle
```

### Why needed:

- The kernel may still have the old partition table cached
- `udevadm settle` forces the kernel to re-read the partition table

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View block devices | `lsblk -fp` |
| View UUIDs | `blkid /dev/sdd1` |
| Create GPT partition (interactive) | `cfdisk /dev/sdd` |
| Create MBR partition (interactive) | `fdisk /dev/sdd` |
| Create file system (XFS) | `mkfs.xfs /dev/sdd1` |
| Create file system (EXT4) | `mkfs.ext4 /dev/sdd2` |
| Create mount point | `mkdir -p /mnt/data` |
| Mount temporarily | `mount /dev/sdd1 /mnt/data` |
| Unmount | `umount /mnt/data` |
| Test fstab | `mount -av` |
| Force partition table re-read | `udevadm settle` |
| Edit fstab | `vi /etc/fstab` |
| Install cfdisk/fdisk | `dnf install -y util-linux` |
| Install Cockpit storage | `dnf install -y cockpit-storaged` |

---

# Partitioning Tools — Quick Reference

| Tool | Command | Best For |
|------|---------|----------|
| cfdisk | `cfdisk /dev/sdd` | Interactive, NCurses (recommended) |
| fdisk | `fdisk /dev/sdd` | Traditional, interactive |
| parted | `parted -s /dev/sdd mklabel gpt mkpart primary 0% 100%` | Scripting (complex) |
| Cockpit | Web interface | GUI, easiest |

---

# Key Takeaways

- **Partitioning** = dividing a block device into separate units.
- **GPT** is modern — supports >2TB disks and 128 partitions. **MBR** is legacy.
- **Primary partitions** can receive file systems.
- **Extended partitions** cannot receive file systems — they host **logical** partitions.
- **Extended partitions always get number 5** (1-4 reserved for primary partitions).
- **Use `lsblk -fp`** to view block devices, partitions, file systems, and UUIDs.
- **Recommended partitioning tools:** `cfdisk` (interactive) or **Cockpit** (GUI).
- **Avoid `parted`** unless necessary — it's unnecessarily complex.
- **Always use UUIDs in `/etc/fstab`** — device names can change.
- **Test `/etc/fstab`** with `mount -av` **before rebooting** — syntax errors can cause maintenance mode.
- **`udevadm settle`** forces the kernel to re-read the partition table after partitioning.