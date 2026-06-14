# Mounting and Unmounting File Systems — Notes

## 1. Core Concepts

| Term | Definition |
|------|------------|
| **Mount** | Make a file system available |
| **Mount point** | Directory where the file system is made available |
| **Unmount** | Detach a mounted file system |

> 💡 You need a **file system** on a block device before you can mount it.

---

## 2. Creating a File System — `mkfs`

### Check if a block device has a file system:

```bash
lsblk -fp
```

| Option | Meaning |
|--------|---------|
| `-f` | Show file system information |
| `-p` | Show full device paths |

### Create an EXT4 file system:

```bash
mkfs.ext4 /dev/sdb
```

### Create an XFS file system (RHEL default):

```bash
mkfs -t xfs /dev/sdc
```

or

```bash
mkfs.xfs /dev/sdc
```

### What `mkfs` does:

- Writes a file system to the block device
- Prints the file system's **UUID** (or you can find it with `lsblk -fp`)

> 💡 **Do you need to partition first?** Not necessarily. You can put a file system directly on a block device (`/dev/sdb`) without partitioning.

---

## 3. Mounting a File System — `mount`

### Basic mount (using device file — can be unreliable):

```bash
mount /dev/sdb /vms
```

### Better: Mount by UUID (reliable):

```bash
mount UUID=3a9b1234-5678-90ab-cdef-1234567890ab /isos
```

### Find the UUID:

```bash
lsblk -fp
```

**Output snippet:**
```
/dev/sdb ext4 3a9b1234-5678-90ab-cdef-1234567890ab /vms
```

### Mount point must exist:

```bash
mkdir -p /vms
mkdir -p /isos
```

> ⚠️ **Important:** Mounts done with the `mount` command are **not persistent** — they disappear after reboot.

---

## 4. Persistent Mounts — `/etc/fstab`

- **`/etc/fstab`** = file system table
- Controls which file systems are mounted **automatically at boot**

| Location | Meaning |
|----------|---------|
| `/etc/` | Configuration directory |
| `fstab` | File system table |

> 📌 Persistent mounting is covered in the RH134 course (System Administration II).

---

## 5. Verifying Mounts — `lsblk -fp`

```bash
lsblk -fp
```

**Example output:**
```
NAME     FSTYPE UUID                                 MOUNTPOINT
/dev/sdb ext4  3a9b1234-5678-90ab-cdef-1234567890ab /vms
/dev/sdc xfs   abcdef12-3456-7890-abcd-ef1234567890 /isos
```

---

## 6. Unmounting — `umount` (not `unmount`!)

### Two ways to unmount:

| Method | Command |
|--------|---------|
| By mount point | `umount /vms` |
| By device file | `umount /dev/sdc` |

> ⚠️ Spelling: **`umount`** (no 'n') — common source of errors.

---

## 7. The "Target is Busy" Problem

### What happens:

```bash
umount /isos
umount: /isos: target is busy
```

### Why:

- A process is currently using files/directories **below the mount point**

### Find the offending process — `lsof`:

```bash
lsof /isos
```

**Example output:**
```
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash    220436 root  cwd    DIR   8,32     4096    2 /isos
```

| Column | Meaning |
|--------|---------|
| `COMMAND` | Command using the mount point |
| `PID` | Process ID |
| `USER` | User running the process |

### Solution:

1. Change out of the mount point directory:

```bash
cd /
```

or

```bash
cd ~
```

2. Try `umount` again

3. Verify no processes:

```bash
lsof /isos    # Should return nothing
```

> 💡 You cannot unmount a file system that is currently in use.

---

## 8. To Partition or Not to Partition?

### Argument against partitioning (when not needed):

| Pro | Con |
|-----|-----|
| Simpler — use entire disk | Someone might accidentally overwrite the file system |

### Counter-argument:

> *"If someone accidentally overwrites your file system, you're hiring the wrong people."*

### Safety:

- `mkfs` commands will **warn** if a file system already exists
- Don't partition unless you need to **divide** the storage

> 💡 Partitioning is covered in RH134 (System Administration II).

---

## 9. Complete Workflow Example

### Step 1: Create file system

```bash
mkfs.ext4 /dev/sdb
```

### Step 2: Create mount point

```bash
mkdir -p /vms
```

### Step 3: Mount the file system

```bash
mount /dev/sdb /vms
```

### Step 4: Verify

```bash
lsblk -fp
```

### Step 5: Use the space

```bash
touch /vms/testfile
df -h /vms
```

### Step 6: Unmount when done

```bash
cd /                # Change out of /vms first
umount /vms
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Create EXT4 file system | `mkfs.ext4 /dev/sdb` |
| Create XFS file system | `mkfs -t xfs /dev/sdc` or `mkfs.xfs /dev/sdc` |
| Mount by device file | `mount /dev/sdb /vms` |
| Mount by UUID | `mount UUID=<uuid> /isos` |
| List block devices with file systems | `lsblk -fp` |
| Find UUID of a file system | `lsblk -fp /dev/sdb` |
| Unmount by mount point | `umount /vms` |
| Unmount by device file | `umount /dev/sdb` |
| Find processes using a mount point | `lsof /isos` |
| Create mount point directory | `mkdir -p /vms` |
| Change out of busy mount point | `cd /` |

---

# Mount Types Summary

| Type | Command | Persistence | Use Case |
|------|---------|-------------|----------|
| Temporary | `mount` | Lost on reboot | Testing, one-time access |
| Persistent | `/etc/fstab` entry | Survives reboot | Production, permanent storage |

> 📌 `/etc/fstab` is covered in RH134.

---

# Troubleshooting: "Target is Busy"

```
┌─────────────────────────────────────────────────────────┐
│ 1. User tries to unmount /isos                          │
│              ↓                                          │
│ 2. umount: target is busy                               │
│              ↓                                          │
│ 3. Run: lsof /isos                                      │
│              ↓                                          │
│ 4. Identifies: bash (PID 220436) is inside /isos       │
│              ↓                                          │
│ 5. User runs: cd /                                      │
│              ↓                                          │
│ 6. Run: lsof /isos (no output)                          │
│              ↓                                          │
│ 7. umount /isos → SUCCESS                               │
└─────────────────────────────────────────────────────────┘
```

---

# Key Takeaways

- **Mount** = make a file system available at a **mount point** (directory)
- **`mkfs.ext4`** or **`mkfs -t xfs`** creates a file system on a block device
- **You don't always need partitions** — file system can go directly on the block device
- **`lsblk -fp`** shows file systems, UUIDs, and mount points
- **Mount by UUID** is more reliable than mounting by device file name (`/dev/sdb`)
- **Temporary mounts** (`mount` command) disappear after reboot
- **Persistent mounts** require `/etc/fstab` (covered in RH134)
- **`umount`** (no 'n') — remember the spelling!
- **`lsof`** helps find what's keeping a mount point busy
- Change out of a mount point directory **before** trying to unmount
- Partitioning tools (`fdisk`, `gdisk`, `parted`) are covered in System Administration II