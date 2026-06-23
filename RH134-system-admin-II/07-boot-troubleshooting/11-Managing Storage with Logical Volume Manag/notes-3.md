# Replacing Physical Volumes and Managing LVM — Notes

## 1. The Scenario — Replacing a Failing Disk

- `/dev/sdc` is old, slow, or failing
- `/dev/sdd` is the replacement disk
- Data on `/dev/sdc` must be moved before removal

### Workflow Overview:

```
1. Add new disk (sdd) to volume group
2. Move extents from old disk (sdc) to new disk (sdd)
3. Remove old disk (sdc) from volume group
4. Physically replace the disk
```

---

## 2. Step 1 — Extend Volume Group with New Disk

### Add `/dev/sdd` to the volume group:

```bash
vgextend vg_data /dev/sdd
```

**What happens:**
- `/dev/sdd` is automatically initialized as a physical volume
- It is added to `vg_data`

### Verify:

```bash
pvs
vgs
```

---

## 3. Step 2 — Move Extents from Old Disk to New Disk

### Move all extents from `/dev/sdc` to `/dev/sdd`:

```bash
pvmove /dev/sdc /dev/sdd
```

### Move extents without specifying destination:

```bash
pvmove /dev/sdc
```

**Result:** Extents are moved to any available physical volume in the VG.

### Verify:

```bash
lsblk -fp
pvs
```

**Expected output after pvmove:**
- `/dev/sdc` has **no data extents**
- `/dev/sdd` now contains the data

---

## 4. Step 3 — Remove Old Disk from Volume Group

### Reduce the volume group:

```bash
vgreduce vg_data /dev/sdc
```

### Verify:

```bash
pvs
```

**Output:**
```
PV         VG       Fmt  Attr PSize PFree
/dev/sdb   vg_data  lvm2 a--  4.99G 1.99G
/dev/sdc            lvm2 ---  5.00G 5.00G   ← Not in a VG
/dev/sdd   vg_data  lvm2 a--  5.00G 2.00G
```

> 💡 `/dev/sdc` is now **not** part of any volume group. The disk can be physically removed.

---

## 5. Removing a Logical Volume — `lvremove`

### Before removing:

1. **Unmount** the file system:

```bash
umount /var/lib/mysql
```

2. **Remove** the entry from `/etc/fstab`

### Remove the logical volume:

```bash
lvremove vg_data/lv_mysql
```

> 💡 Always specify the volume group name — logical volumes with the same name could exist in different VGs.

### Verify:

```bash
lvs
```

---

## 6. LVM Backup and Restore — `vgcfgrestore`

### Why this matters:

- The **volume group** is central to LVM
- Every change to the VG is **backed up** automatically
- You can **restore** to a previous state

### View restore points:

```bash
vgcfgrestore -l vg_data
```

**Output example:**
```
File:         /etc/lvm/backup/vg_data
  VG name:     vg_data
  Description: Created *before* executing 'vgreduce'
  Backup Time: Tue Sep  1 14:30:00 2025

File:         /etc/lvm/backup/vg_data
  VG name:     vg_data
  Description: Created *before* executing 'lvremove'
  Backup Time: Tue Sep  1 14:35:00 2025
```

### Restore to a previous state:

```bash
vgcfgrestore -f /etc/lvm/backup/vg_data vg_data
```

### Use cases:

| Scenario | Restore Point |
|----------|---------------|
| Accidentally removed a PV | Restore before `vgreduce` |
| Accidentally removed an LV | Restore before `lvremove` |

---

## 7. LVM Commands for Disk Replacement

| Task | Command |
|------|---------|
| Add new disk to VG | `vgextend vg_data /dev/sdd` |
| Move extents from old disk | `pvmove /dev/sdc /dev/sdd` |
| Remove old disk from VG | `vgreduce vg_data /dev/sdc` |
| Remove logical volume | `lvremove vg_data/lv_mysql` |
| View restore points | `vgcfgrestore -l vg_data` |
| Restore VG to previous state | `vgcfgrestore -f /etc/lvm/backup/vg_data vg_data` |

---

## 8. Disk Replacement — Visual Flow

```
┌─────────────────────────────────────────────────────────────┐
│ Initial State                                               │
│ ┌───────┐  ┌───────┐  ┌───────┐                           │
│ │ /dev/sdb│  │ /dev/sdc│  │ /dev/sdd│ (empty)              │
│ │  (PV)   │  │  (PV)   │  │         │                      │
│ └────┬────┘  └────┬────┘  └─────────┘                      │
│      │            │                                         │
│      └────────────┼─────────────────────────────────────────┘
│                   │                                          │
│              vg_data                                         │
└─────────────────────────────────────────────────────────────┘

After vgextend:
┌───────┐  ┌───────┐  ┌───────┐
│ /dev/sdb│  │ /dev/sdc│  │ /dev/sdd│  ← Added
│  (PV)   │  │  (PV)   │  │  (PV)   │
└────┬────┘  └────┬────┘  └────┬────┘
     │            │            │
     └────────────┼────────────┘
                  │
              vg_data

After pvmove:
┌───────┐  ┌───────┐  ┌───────┐
│ /dev/sdb│  │ /dev/sdc│  │ /dev/sdd│  ← Data moved here
│  (PV)   │  │  (empty)│  │  (PV)   │
└────┬────┘  └────┬────┘  └────┬────┘
     │            │            │
     └────────────┼────────────┘
                  │
              vg_data

After vgreduce:
┌───────┐  ┌───────┐  ┌───────┐
│ /dev/sdb│  │ /dev/sdc│  │ /dev/sdd│
│  (PV)   │  │(removed)│  │  (PV)   │
└────┬────┘  └─────────┘  └────┬────┘
     │                          │
     └──────────┬───────────────┘
                │
            vg_data
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Add PV to VG | `vgextend vg_data /dev/sdd` |
| Move extents (with destination) | `pvmove /dev/sdc /dev/sdd` |
| Move extents (auto destination) | `pvmove /dev/sdc` |
| Remove PV from VG | `vgreduce vg_data /dev/sdc` |
| Remove LV | `lvremove vg_data/lv_mysql` |
| Unmount filesystem | `umount /var/lib/mysql` |
| View restore points | `vgcfgrestore -l vg_data` |
| Restore VG | `vgcfgrestore -f /etc/lvm/backup/vg_data vg_data` |
| View PVs | `pvs` |
| View VGs | `vgs` |
| View LVs | `lvs` |
| View block devices | `lsblk -fp` |

---

# Key Takeaways

- **`vgextend`** adds a new physical volume to a volume group.
- **`pvmove`** moves extents from one PV to another — **online, no downtime**.
- **`vgreduce`** removes a physical volume from a volume group (after data is moved).
- **Always move data** before removing a physical volume.
- **Backups are automatic** — every LVM change creates a restore point.
- **`vgcfgrestore -l`** lists available restore points.
- **`vgcfgrestore`** can restore a volume group to a previous state.
- **Remove LV** before removing the VG if the LV is no longer needed.
- **Always unmount** file systems before removing logical volumes.