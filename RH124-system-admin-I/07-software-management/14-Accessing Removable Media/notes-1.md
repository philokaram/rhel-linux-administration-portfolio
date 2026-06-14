# Identifying File Systems and Block Devices — Notes

## 1. What Is a Block Device?

- **Block device** = storage device (hard drive, SSD, USB stick)
- Need a **file system** to organize files on a block device

### Windows vs Linux — Comparison

| Aspect | Windows | Linux |
|--------|---------|-------|
| File system | NTFS | XFS (default), EXT4, exFAT |
| Access method | Drive letters (C:\, D:\) | **Mount points** (directories) |
| Device representation | Hidden | `/dev/` directory (device files) |

---

## 2. Block Device Naming in Linux

Device files live in `/dev/` and **don't occupy space**.

### Naming conventions by storage type:

| Storage Type | Device File Pattern | Example |
|--------------|--------------------|---------|
| SATA, SAS, SCSI, USB | `/dev/sdX` | `/dev/sda`, `/dev/sdb`, `/dev/sdc` |
| Virtual disks (KVM, Xen) | `/dev/vdX` | `/dev/vda`, `/dev/vdb` |
| NVMe (SSD) | `/dev/nvme0n1` | `/dev/nvme0n1`, `/dev/nvme1n1` |

### Partitions:

| Device | Partition | Device File |
|--------|-----------|-------------|
| First SATA disk | Partition 1 | `/dev/sda1` |
| First SATA disk | Partition 2 | `/dev/sda2` |
| First SATA disk | Partition 3 | `/dev/sda3` |

> 💡 Partitions are numbered: `sda1`, `sda2`, `sda3`, etc.

---

## 3. Viewing Block Devices — `lsblk`

### Basic command:

```bash
lsblk
```

### Better — with full paths and file system info:

```bash
lsblk -fp
```

| Option | Meaning |
|--------|---------|
| `-f` | Show file system information |
| `-p` | Show full device paths |

### Example output:

```
NAME         FSTYPE LABEL UUID                                 MOUNTPOINT
/dev/sda                                                      
├─/dev/sda1                                                   
├─/dev/sda2  vfat        1234-5678                            /boot/efi
└─/dev/sda3  xfs          abcdef12-3456-7890-abcd-ef1234567890 /
/dev/sdb                                                      
/dev/sdc                                                      
/dev/sdd                                                      
```

### What `lsblk -fp` shows:

| Column | Example | Meaning |
|--------|---------|---------|
| NAME | `/dev/sda3` | Device file path |
| FSTYPE | `xfs` | File system type |
| LABEL | (optional) | Human-readable label |
| UUID | `abcd...` | Universally Unique Identifier |
| MOUNTPOINT | `/` | Where it's mounted |

> 💡 **UUID** is the **reliable** way to identify file systems (not device names like `/dev/sda3`).

---

## 4. Mounting — Making File Systems Available

### What is mounting?

**Mounting** = the process of making a file system available at a **mount point**.

- **Mount point** = a directory
- Once mounted, you access the storage by going to that directory

### Windows vs Linux mounting:

| Windows | Linux |
|---------|-------|
| C:\ | `/` (root) |
| D:\ | `/mnt/data` |
| E:\ | `/home` |

### Example hierarchy:

```
/ (root)           ← mounted from /dev/sda3
├── etc/           ← part of root
├── usr/           ← could be separate partition (/dev/sda4)
├── boot/efi/      ← mounted from /dev/sda2
└── mnt/data/      ← mounted from /dev/sdb (entire disk)
```

### Key concept:

- If `/` fills up → system has problems
- Separate mount points isolate storage (e.g., `/usr` filling up doesn't affect `/`)

---

## 5. Viewing Mounted File Systems — `mount` and `df`

### `mount` command (verbose):

```bash
mount
```

**Shows:** All mounted file systems (including virtual/special file systems)

> 📌 Can be overwhelming — lots of virtual file systems.

### `df` command (disk free) — better for usage:

```bash
df -h
```

| Option | Meaning |
|--------|---------|
| `-h` | Human-readable (KB, MB, GB) |

**Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3       9.8G  3.9G  5.9G  40% /
/dev/sda2       200M  8.4M  192M   5% /boot/efi
```

---

## 6. Directory Size — `du`

```bash
du -sh /etc
du -sh /usr
```

| Option | Meaning |
|--------|---------|
| `-s` | Summary (total size, not per file) |
| `-h` | Human-readable |

**Example output:**
```
25M    /etc
2.8G   /usr
```

> 💡 `/usr` (software warehouse) often consumes significant space.

---

## 7. The Root File System — Why Size Matters

### Kernel update behavior:

- **Don't replace** the current kernel
- **Install additional kernel** alongside old ones
- Multiple kernels can accumulate

| Number of kernels | Space usage |
|-------------------|-------------|
| 1 kernel | Baseline |
| 2 kernels | More space |
| 3 kernels | Even more space |

> 💡 Ensure `/boot` and `/boot/efi` have enough space for multiple kernels.

### Example `/boot/efi` size:

```
200M total, 8.4M used, 192M available
```

---

## 8. Bytes, Kibibytes, Megabytes — The Messy Truth

### The origin story:

- Old spinning disks had **sectors** of **512 bytes**
- Actuating arm could read **top and bottom** = 1024 bytes at a time
- 1024 bytes was easy to call "1 Kilobyte" (but it's not!)

### Binary vs Decimal prefixes:

| Unit | Binary (Base 2) | Decimal (Base 10) | Difference |
|------|-----------------|-------------------|------------|
| 1 KB vs 1 KiB | 2¹⁰ = 1,024 bytes | 10³ = 1,000 bytes | 24 bytes |
| 1 MB vs 1 MiB | 2²⁰ = 1,048,576 bytes | 10⁶ = 1,000,000 bytes | 48,576 bytes |
| 1 GB vs 1 GiB | 2³⁰ = 1,073,741,824 bytes | 10⁹ = 1,000,000,000 bytes | 73,741,824 bytes |
| 1 TB vs 1 TiB | 2⁴⁰ = 1,099,511,627,776 bytes | 10¹² = 1,000,000,000,000 bytes | 99,511,627,776 bytes |

### Who uses what:

| Manufacturer/Software | Prefix | Example |
|-----------------------|--------|---------|
| Hard drive vendors | Decimal (base 10) | "4 TB" = 4,000,000,000,000 bytes |
| Memory subsystem | Binary (base 2) | "4 GB" RAM = 4,294,967,296 bytes |
| Linux tools (`df`, `lsblk`) | Binary (GiB, MiB) | Shows "9.8G" = 9.8 GiB |

### The standard (since 1998):

- **IEC standard** binary prefixes: KiB, MiB, GiB, TiB
- **MB** = million bytes (decimal)
- **MiB** = 1,048,576 bytes (binary)

### View prefix definitions:

```bash
man units
```

### Quote from man page:

> *"Thus today, MB equals a million bytes, and therefore MiB equals 1048576 bytes."*

---

## 9. The Tree Analogy — Revisited

```
/ (root)
├── etc/          ← part of root file system
├── usr/          ← could be separate mount point
├── boot/efi/     ← separate file system (vfat)
├── mnt/          ← could be separate disk/partition
└── data/         ← separate partition (/dev/sdc1)
```

> 💡 Different mount points = different storage locations.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| List block devices (basic) | `lsblk` |
| List block devices with file system info | `lsblk -fp` |
| Show mounted file systems | `mount` |
| Show disk usage (human-readable) | `df -h` |
| Show directory size | `du -sh /path` |
| View device files | `ls /dev/sd* /dev/vd*` |
| Check if device has file system | `lsblk -fp /dev/sdb` |
| View prefix definitions | `man units` |

---

# File System Types in RHEL

| File System | Default? | Use Case |
|-------------|----------|----------|
| **XFS** | ✅ Yes | General purpose, default |
| **EXT4** | No | Alternative, good for smaller files |
| **exFAT** | No | USB drives, cross-platform |
| **vfat** | No | `/boot/efi` (UEFI systems) |

---

# Device Naming Cheat Sheet

| Storage Type | Pattern | First Device | First Partition |
|--------------|---------|--------------|-----------------|
| SATA/SAS/SCSI/USB | `sdX` | `/dev/sda` | `/dev/sda1` |
| Virtual disk | `vdX` | `/dev/vda` | `/dev/vda1` |
| NVMe | `nvme0n1` | `/dev/nvme0n1` | `/dev/nvme0n1p1` |

---

# Key Takeaways

- **Block device** = storage device (hard drive, SSD, etc.)
- **Device files** are in `/dev/` and don't occupy space
- **Naming convention** depends on storage type: `sdX`, `vdX`, `nvme0n1`
- **Partitions** are numbered: `/dev/sda1`, `/dev/sda2`, etc.
- **`lsblk -fp`** is the best command to see block devices, file systems, and mount points
- **UUID** is the reliable way to identify file systems (not device names)
- **Mounting** = making a file system available at a **mount point** (directory)
- **`df -h`** shows disk usage; **`du -sh`** shows directory size
- **Binary prefixes** (KiB, MiB, GiB) use base 2 (powers of 1024)
- **Decimal prefixes** (KB, MB, GB) use base 10 (powers of 1000)
- Hard drive vendors use decimal; Linux tools typically use binary
- **`man units`** explains prefix differences in detail