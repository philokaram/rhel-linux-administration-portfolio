# Extending Logical Volumes — Notes

## 1. Extending a Volume Group — `vgextend`

### The problem:

- Volume group is running out of space
- Can't create new logical volumes
- Can't grow existing logical volumes

### The solution — Add a new physical volume:

```bash
vgextend vg_data /dev/sdc
```

**What happens:**
1. `/dev/sdc` is automatically initialized as a physical volume
2. The physical volume is added to the volume group

### Verify:

```bash
pvs
vgs
```

### Before vs After:

| Scenario | VG Size | Free Space |
|----------|---------|------------|
| Before (`/dev/sdb` only) | ~5 GiB | ~3 GiB |
| After (added `/dev/sdc`) | ~10 GiB | ~8 GiB |

> 💡 `vgextend` automatically runs `pvcreate` — no need to run it separately.

---

## 2. Extending Logical Volumes — Three Approaches

| Approach | Command | File System Resized? |
|----------|---------|---------------------|
| **Two commands** | `lvextend` + `xfs_growfs` | ✅ Yes (separate) |
| **Two commands** | `lvextend` + `resize2fs` | ✅ Yes (separate) |
| **One command** | `lvresize -r` | ✅ Yes (automatic) |

> 💡 **Recommendation:** Use `lvresize -r` — one command does it all.

---

## 3. Extending XFS — Two-Command Approach

### Step 1 — Extend the logical volume:

```bash
lvextend -L 6G /dev/vg_data/lv_docs
```

### Step 2 — Extend the file system:

```bash
xfs_growfs /mnt/docs
```

> 💡 XFS can only be **grown**, not shrunk.

---

## 4. Extending EXT4 — Two-Command Approach

### Step 1 — Extend the logical volume:

```bash
lvextend -L 3G /dev/vg_data/lv_mysql
```

### Step 2 — Extend the file system:

```bash
resize2fs /dev/vg_data/lv_mysql
```

> 💡 EXT4 can be **grown** online and **shrunk** offline.

---

## 5. One-Command Approach — `lvresize -r`

### Resize logical volume AND file system together:

```bash
lvresize -r -L 6G /dev/vg_data/lv_docs
```

| Option | Meaning |
|--------|---------|
| `-r` | Resize the file system as well |
| `-L` | New size (absolute) |

### Verify:

```bash
df -h /mnt/docs
lvs
```

> 💡 This is the **recommended** approach — fewer commands, less chance of error.

---

## 6. Resizing with `+` — Adding Space

### Add 1 GiB to the logical volume:

```bash
lvresize -r -L +1G /dev/vg_data/lv_mysql
```

### Use a percentage of free space:

```bash
lvresize -r -l +50%FREE /dev/vg_data/lv_mysql
```

| Option | Meaning |
|--------|---------|
| `-l` | Use physical extents (percentage) |
| `+50%FREE` | Add 50% of the free space in the VG |

---

## 7. Creating a New Logical Volume with EXT4

### Create LV:

```bash
lvcreate -n lv_mysql -L 2G vg_data
```

### Create EXT4 file system:

```bash
mkfs.ext4 /dev/vg_data/lv_mysql
```

### Get UUID:

```bash
blkid /dev/vg_data/lv_mysql
```

### Mount persistently:

```
UUID=<uuid> /var/lib/mysql ext4 defaults 0 0
```

### Test:

```bash
mount -av
```

---

## 8. Shrinking EXT4 — `lvresize -r` (Offline)

### Step 1 — Unmount the file system:

```bash
umount /var/lib/mysql
```

### Step 2 — Resize (logical volume + file system):

```bash
lvresize -r -L 1G /dev/vg_data/lv_mysql
```

### Step 3 — Remount:

```bash
mount -av
```

> ⚠️ **Shrinking requires unmounting** (EXT4) and is **not supported** for XFS.

---

## 9. LVM for Swap

### Create swap logical volume:

```bash
lvcreate -n lv_swap -L 2G vg_data
```

### Create swap file system:

```bash
mkswap /dev/vg_data/lv_swap
```

### Activate swap:

```bash
swapon /dev/vg_data/lv_swap
```

### Persistent entry:

```
/dev/vg_data/lv_swap none swap defaults 0 0
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Extend volume group | `vgextend vg_data /dev/sdc` |
| Extend LV (absolute) | `lvextend -L 6G /dev/vg_data/lv_docs` |
| Extend LV (add space) | `lvextend -L +1G /dev/vg_data/lv_docs` |
| Extend XFS file system | `xfs_growfs /mnt/docs` |
| Extend EXT4 file system | `resize2fs /dev/vg_data/lv_mysql` |
| One-command resize | `lvresize -r -L 6G /dev/vg_data/lv_docs` |
| One-command resize (+ space) | `lvresize -r -L +1G /dev/vg_data/lv_mysql` |
| Use free space percentage | `lvresize -r -l +50%FREE /dev/vg_data/lv_mysql` |
| Create LV | `lvcreate -n lv_mysql -L 2G vg_data` |
| Create EXT4 filesystem | `mkfs.ext4 /dev/vg_data/lv_mysql` |
| Create swap LV | `lvcreate -n lv_swap -L 2G vg_data` |
| Create swap filesystem | `mkswap /dev/vg_data/lv_swap` |
| Activate swap | `swapon /dev/vg_data/lv_swap` |
| View LVs | `lvs` |
| View VGs | `vgs` |
| View PVs | `pvs` |
| View mount info | `df -h /mnt/docs` |

---

# Resizing — XFS vs EXT4

| Feature | XFS | EXT4 |
|---------|-----|------|
| **Grow online** | ✅ Yes (`xfs_growfs`) | ✅ Yes (`resize2fs`) |
| **Shrink online** | ❌ No | ❌ No |
| **Shrink offline** | ❌ No | ✅ Yes (`resize2fs`) |
| **Recommended resize tool** | `lvresize -r` | `lvresize -r` |

---

# LVM Resizing — Visual Comparison

### Two-command approach:

```
lvextend -L 6G /dev/vg_data/lv_docs
         │
         ▼
LV is 6 GiB, file system still 4 GiB
         │
         ▼
xfs_growfs /mnt/docs
         │
         ▼
LV and filesystem both 6 GiB ✅
```

### One-command approach:

```
lvresize -r -L 6G /dev/vg_data/lv_docs
         │
         ▼
LV and filesystem both 6 GiB ✅
```

---

# Key Takeaways

- **`vgextend`** adds physical volumes to a volume group (auto-runs `pvcreate`).
- **`lvextend`** extends the logical volume (LV) only.
- **`xfs_growfs`** extends XFS file systems (online).
- **`resize2fs`** extends EXT4 file systems (online).
- **`lvresize -r`** extends BOTH the logical volume and the file system — one command.
- **Use `+`** to add space (e.g., `-L +1G`).
- **Use `%FREE`** to use a percentage of free space in the VG.
- **XFS cannot be shrunk** — only grown.
- **EXT4 can be shrunk** but requires **unmounting**.
- **For swap on LVM**, resizing must be done **offline** (swapoff first).