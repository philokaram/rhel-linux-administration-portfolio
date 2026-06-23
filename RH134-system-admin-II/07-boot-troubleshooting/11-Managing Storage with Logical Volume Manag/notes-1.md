# Creating Logical Volumes — Notes

## 1. The Problem with Traditional Storage

### Traditional approach:

```
Block Device (/dev/sdb) → File System → Mount Point
```

**Problem:** If the file system fills up:
1. Back up the data
2. Buy a new disk
3. Create a new file system
4. Restore the data
5. Downtime required

> 💡 **LVM solves this** by allowing resizable, flexible storage without downtime.

---

## 2. LVM Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Block Devices                             │
│              /dev/sdb    /dev/sdc    /dev/sdd               │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                   Physical Volumes (PV)                     │
│               /dev/sdb    /dev/sdc    /dev/sdd              │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                    Volume Group (VG)                        │
│                        vg_data                               │
│                  (Pool of storage)                          │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                   Logical Volumes (LV)                      │
│              lv_data    lv_docs    lv_backup                │
└─────────────────────────────────────────────────────────────┘
```

### Layers:

| Layer | Description | Example |
|-------|-------------|---------|
| **Physical Volume (PV)** | Raw block device added to LVM | `/dev/sdb` |
| **Volume Group (VG)** | Pool of storage, spans multiple disks | `vg_data` |
| **Logical Volume (LV)** | Flexible unit of storage | `lv_data` |

---

## 3. LVM Commands — Quick Reference

| Task | Command |
|------|---------|
| Create Physical Volume | `pvcreate /dev/sdb` (optional) |
| Create Volume Group | `vgcreate vg_data /dev/sdb` |
| Create Logical Volume | `lvcreate -n lv_data -L 2G vg_data` |
| List Physical Volumes | `pvs` |
| List Volume Groups | `vgs` |
| List Logical Volumes | `lvs` |
| Rename Logical Volume | `lvrename vg_data lv_data lv_docs` |
| Extend Logical Volume | `lvextend -L +1G /dev/vg_data/lv_data` |
| Extend file system (XFS) | `xfs_growfs /mnt/docs` |
| Extend file system (EXT4) | `resize2fs /dev/vg_data/lv_data` |

---

## 4. Creating LVM — Minimal Steps

### Step 1 — Create Volume Group (VG):

```bash
vgcreate vg_data /dev/sdb
```

**What happens:**
- `pvcreate` is automatically run on `/dev/sdb`
- The volume group is created

### Step 2 — Create Logical Volume (LV):

```bash
lvcreate -n lv_data -L 2G vg_data
```

| Option | Meaning |
|--------|---------|
| `-n` | Name of the logical volume |
| `-L` | Size (e.g., `2G`, `500M`) |

### Step 3 — Create file system:

```bash
mkfs.xfs /dev/vg_data/lv_data
```

### Step 4 — Mount:

```bash
mkdir -p /mnt/data
mount /dev/vg_data/lv_data /mnt/data
```

---

## 5. Device File Naming

### Two ways to reference a logical volume:

| Notation | Example |
|----------|---------|
| `/dev/volume_group/logical_volume` | `/dev/vg_data/lv_data` |
| `/dev/mapper/volume_group-logical_volume` | `/dev/mapper/vg_data-lv_data` |

> 💡 Both refer to the same logical volume.

---

## 6. Viewing LVM Information

### Physical Volumes:

```bash
pvs
```

**Output:**
```
PV         VG       Fmt  Attr PSize PFree
/dev/sdb   vg_data  lvm2 a--  4.99G 2.99G
```

### Volume Groups:

```bash
vgs
```

**Output:**
```
VG       #PV #LV #SN Attr   VSize  VFree
vg_data    1   1   0 wz--n- 4.99G 2.99G
```

### Logical Volumes:

```bash
lvs
```

**Output:**
```
LV       VG       Attr       LSize  Pool Origin Data%
lv_data  vg_data  -wi-a----- 2.00g
```

---

## 7. Physical Extents (PE)

- The **smallest unit** of storage in LVM
- Size is a property of the **volume group**
- Default PE size = **4 MiB**

### View PE details:

```bash
vgdisplay vg_data
```

### Specify PE size when creating VG:

```bash
vgcreate -s 8M vg_data /dev/sdb
```

> 💡 PE size must be a power of 2 (e.g., `4M`, `8M`, `16M`).

---

## 8. Renaming Logical Volumes — `lvrename`

### Before:

```bash
lvcreate -n lv_data -L 2G vg_data
```

### Rename:

```bash
lvrename vg_data lv_data lv_docs
```

### Verify:

```bash
lvs
```

---

## 9. Persistent Mounting — `/etc/fstab`

### Get UUID:

```bash
blkid /dev/vg_data/lv_data
```

or

```bash
lsblk -fp /dev/vg_data/lv_data
```

### Add to `/etc/fstab`:

```
UUID=<uuid> /mnt/docs xfs defaults 0 0
```

### Test:

```bash
mount -av
```

### Verify:

```bash
lsblk -fp
```

---

## 10. Web Console — LVM Management

### Can be managed via Cockpit:

- **Create LV:** ✅ Yes (from existing VG)
- **Grow LV:** ✅ Yes
- **Deactivate LV:** ✅ Yes
- **Delete LV:** ✅ Yes
- **Add PV to VG:** ✅ Yes
- **Create VG:** ❌ No (use command line)

> 💡 Use the web console to **manage** existing LVM, but **create** with command line.

---

## 11. Cleaning Up Old LVM — `dd` to Overwrite

### Overwrite partition table (first 100 MiB):

```bash
dd if=/dev/zero of=/dev/sdb bs=1M count=100
```

### Loop for multiple devices:

```bash
for i in b c d; do
    dd if=/dev/zero of=/dev/sd$i bs=1M count=100
done
```

> ⚠️ **Warning:** This permanently destroys data on the device.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Create VG (and PV automatically) | `vgcreate vg_data /dev/sdb` |
| Create LV | `lvcreate -n lv_data -L 2G vg_data` |
| List PVs | `pvs` |
| List VGs | `vgs` |
| List LVs | `lvs` |
| Rename LV | `lvrename vg_data lv_data lv_docs` |
| Get UUID | `blkid /dev/vg_data/lv_data` |
| Create XFS filesystem | `mkfs.xfs /dev/vg_data/lv_data` |
| Mount | `mount /dev/vg_data/lv_data /mnt/docs` |
| Test fstab | `mount -av` |
| View PE details | `vgdisplay vg_data` |
| Overwrite device | `dd if=/dev/zero of=/dev/sdb bs=1M count=100` |

---

# LVM Layers — Visual Summary

```
/dev/sdb  ──┐
/dev/sdc  ──┼──► Physical Volumes (PV) ──► Volume Group (VG) ──┐
/dev/sdd  ──┘                                                  │
                                                               ▼
                                                      ┌─────────────────┐
                                                      │  Logical Volume │
                                                      │    lv_data      │
                                                      │   (2 GiB)       │
                                                      └─────────────────┘
                                                               │
                                                               ▼
                                                      ┌─────────────────┐
                                                      │  File System    │
                                                      │    XFS          │
                                                      └─────────────────┘
                                                               │
                                                               ▼
                                                      ┌─────────────────┐
                                                      │  Mount Point    │
                                                      │  /mnt/docs      │
                                                      └─────────────────┘
```

---

# Key Takeaways

- **LVM = Logical Volume Management** — flexible, resizable storage.
- **Layers:** PV (physical volumes) → VG (volume groups) → LV (logical volumes).
- **Benefits:**
  - Resizable **on the fly** (no downtime)
  - Spans multiple disks
  - Easy to manage
- **`vgcreate` automatically runs `pvcreate`** — you don't need to run `pvcreate` separately.
- **Device naming:** `/dev/vg_name/lv_name` or `/dev/mapper/vg_name-lv_name`.
- **Physical Extents (PE)** = smallest unit of storage (default 4 MiB).
- **Web Console** can manage existing LVM but **cannot create** VGs.
- **Always use UUID** in `/etc/fstab` for persistent mounts.
- **Test fstab** with `mount -av` before rebooting.
- **`lvrename`** renames logical volumes.
- **Cleaning up:** Use `dd` to overwrite partition tables.