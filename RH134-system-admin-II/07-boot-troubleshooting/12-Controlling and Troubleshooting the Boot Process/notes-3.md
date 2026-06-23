# Repairing Damaged File Systems at Boot — Notes

## 1. The Problem — `/etc/fstab` Errors

During boot, systemd reads `/etc/fstab` and mounts all listed file systems.

### Common errors:

| Error | Cause |
|-------|-------|
| Missing mount point | Directory doesn't exist |
| Typo in UUID | Incorrect UUID |
| Non-existent device | Device file doesn't exist |
| Incorrect file system type | Wrong `fstype` |

> 💡 If any mount fails, the boot process can drop you into **emergency shell**.

---

## 2. Safe Testing — The `nofail` Option

### Use `nofail` to test without breaking boot:

```
/dev/sdx1 /mnt/broken xfs defaults,nofail 0 0
```

### What `nofail` does:

- If the mount fails, systemd **logs the error** but **continues booting**
- The system still comes up normally

### Test with `mount -av`:

```bash
mount -av
```

**Output:**
```
/mnt/broken: fsconfig failed to call /dev/sdx1: No such device
```

> 💡 The error is logged, but the system continues booting.

---

## 3. Breaking Boot — Removing `nofail`

### Without `nofail`:

```
/dev/sdx1 /mnt/broken xfs defaults 0 0
```

- System fails to mount the file system
- Boot process stops
- System drops into **emergency shell**

### Emergency shell prompt:

```
Generating "/run/initramfs/rdsosreport.txt"
Entering emergency mode. Exit the shell to continue.
Type "journalctl" to view system logs.
Give root password for maintenance:
```

---

## 4. Fixing `/etc/fstab` in Emergency Mode

### Step 1 — Enter root password:

```
Give root password for maintenance:
```

### Step 2 — Remount root as read-write:

```bash
mount -o remount,rw /
```

> 💡 Root is mounted as **read-only** in emergency mode.

### Step 3 — Edit `/etc/fstab`:

```bash
vi /etc/fstab
```

- Remove or correct the offending entry

### Step 4 — Reboot:

```bash
reboot
```

---

## 5. Repairing Corrupted File Systems

### Important rule:

> ⚠️ **Never run repair tools on a mounted file system.**

### Unmount the file system first:

```bash
umount /mnt/docs
```

### XFS repair:

```bash
xfs_repair /dev/vg_data/lv_docs
```

### EXT4 repair:

```bash
fsck.ext4 /dev/sdb1
```

### Use `whatis` to find tools:

```bash
whatis xfs_repair
```

**Output:** `xfs_repair (8) - repair an XFS filesystem`

---

## 6. File System Repair — Summary

| File System | Repair Tool | Command |
|-------------|-------------|---------|
| XFS | `xfs_repair` | `xfs_repair /dev/device` |
| EXT4 | `fsck.ext4` | `fsck.ext4 /dev/device` |
| EXT3 | `fsck.ext3` | `fsck.ext3 /dev/device` |
| EXT2 | `fsck.ext2` | `fsck.ext2 /dev/device` |

---

## 7. Emergency Mode — Quick Reference

| Action | Command |
|--------|---------|
| Enter emergency mode | Kernel parameter: `systemd.unit=emergency.target` |
| Root password | Required |
| Remount root as writable | `mount -o remount,rw /` |
| Edit fstab | `vi /etc/fstab` |
| Reboot | `reboot` |

---

## 8. Exam Tip — Test `/etc/fstab` Before Reboot

### Before rebooting:

```bash
umount /mnt/broken   # If currently mounted
mount -av            # Test all fstab entries
```

### Why?

- If `/etc/fstab` has errors, the system may boot into **emergency mode**
- During grading, systems are **rebooted**
- If your system boots into emergency mode → **score 0** for that system

### Strategy:

1. **Test early** — don't wait until the last 10 minutes
2. **Fix errors immediately**
3. **Reboot to verify** — not just `mount -av`

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Test fstab entries | `mount -av` |
| Add nofail option | `defaults,nofail` in `/etc/fstab` |
| Remount root as writable | `mount -o remount,rw /` |
| Edit fstab | `vi /etc/fstab` |
| Unmount file system | `umount /mnt/docs` |
| Repair XFS | `xfs_repair /dev/device` |
| Repair EXT4 | `fsck.ext4 /dev/device` |
| View tool description | `whatis xfs_repair` |
| Reboot system | `reboot` |

---

# Emergency Mode — Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Boot fails (bad fstab entry)                             │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. System drops into emergency shell                        │
│    └── "Give root password for maintenance"                │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Enter root password                                      │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Remount root as writable                                 │
│    └── mount -o remount,rw /                               │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. Fix /etc/fstab (vi)                                      │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. Reboot                                                   │
│    └── System boots normally                                │
└─────────────────────────────────────────────────────────────┘
```

---

# Key Takeaways

- **`/etc/fstab` errors** can prevent the system from booting.
- **Use `nofail`** to test bad entries without breaking boot.
- **Emergency mode** is where you land when a critical mount fails.
- **Root is read-only** in emergency mode — remount with `mount -o remount,rw /`.
- **Fix `/etc/fstab`** and reboot to recover.
- **Never repair** a mounted file system — unmount it first.
- **Repair tools:** `xfs_repair` (XFS), `fsck.ext4` (EXT4).
- **Exam tip:** Reboot early to verify `/etc/fstab` changes — don't wait until the last 10 minutes.