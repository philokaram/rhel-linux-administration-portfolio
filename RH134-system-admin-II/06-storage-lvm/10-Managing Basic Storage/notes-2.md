# Managing Swap Space — Notes

## 1. What Is Swap Space?

- **Swap space** = a backup plan for system RAM
- Temporarily holds **inactive memory pages** (data not currently in use)
- Together with RAM, swap creates the system's **total virtual memory**

### How it works:

```
RAM is full → Kernel moves idle pages to swap → Frees up RAM
                  │
                  ▼
When data is needed → Kernel retrieves from swap → Swaps out inactive page
```

> ⚠️ Swap space is **on disk** — much slower than RAM. It's for handling occasional memory spikes, not a permanent solution.

---

## 2. Swap Space — Recommendations

- See student guide for swap recommendations based on system RAM
- **Avoid sharing** a block device between swap and other workloads → causes disk contention

---

## 3. Viewing Swap Usage — `free` and `swapon`

### View memory + swap (human-readable):

```bash
free -m
```

| Option | Meaning |
|--------|---------|
| `-m` | Show in **MiB** |
| `-h` | Human-readable |

### View swap statistics:

```bash
swapon -s
```

**Shows:**
- Device/file name
- Size
- Used space
- **Priority**

---

## 4. Creating Swap Space — `mkswap`

### On a partition:

```bash
mkswap /dev/sdd2
```

### On an entire block device (no partitions):

```bash
mkswap /dev/sdc
```

### View the UUID:

```bash
lsblk -fp
```

or

```bash
blkid /dev/sdd2
```

---

## 5. Activating Swap — `swapon`

### Activate all swap units:

```bash
swapon -a
```

### Activate a specific swap unit:

```bash
swapon /dev/sdd2
```

### Verify:

```bash
swapon -s
```

---

## 6. Deactivating Swap — `swapoff`

### Deactivate all swap units:

```bash
swapoff -a
```

### Deactivate a specific swap unit:

```bash
swapoff /dev/sdd2
```

---

## 7. Persistent Swap — `/etc/fstab`

### Format:

```
UUID=<uuid> none swap defaults 0 0
```

or with a mount point:

```
UUID=<uuid> swap swap defaults 0 0
```

| Field | Value | Meaning |
|-------|-------|---------|
| 1 | UUID | Unique identifier for the swap device |
| 2 | `none` or `swap` | Swap doesn't have a mount point |
| 3 | `swap` | File system type |
| 4 | `defaults` or `pri=X` | Mount options |
| 5 | `0` | No dump backup |
| 6 | `0` | No fsck |

### Example:

```
UUID=12345678-1234-1234-1234-123456789abc swap swap defaults 0 0
```

---

## 8. Swap Priority — `pri=`

- Higher priority = used first
- Multiple swap units with **same priority** = striped (used in parallel)

### Set priority:

```
UUID=abc... swap swap pri=10 0 0
UUID=def... swap swap pri=10 0 0
UUID=ghi... swap swap pri=5  0 0
```

### How it works:

- All `pri=10` units are used first (striped)
- Then `pri=5` units
- Then system-allocated priorities (negative values)

### System-allocated priority:

```bash
swapon -s
```

**Output:**
```
/dev/sdb2   1.0G  0.0G  -2
```

Negative priority = system-allocated (lower priority than user-defined).

---

## 9. Cockpit — Creating Swap Space (RHEL 10)

### Install storage plugin:

```bash
dnf install -y cockpit-storaged
```

### Steps:

1. Connect to `https://server:9090`
2. Log in (e.g., `student`)
3. Navigate to **Storage**
4. Select a disk
5. Click **Format**
6. Choose **Swap** as the file system type
7. Click **Format and start**

> 💡 Cockpit automatically updates `/etc/fstab` for persistent swap.

---

## 10. Testing Swap Configuration

### Step 1 — Deactivate all swap:

```bash
swapoff -a
```

### Step 2 — Process fstab:

```bash
mount -av
```

### Step 3 — Activate all swap:

```bash
swapon -a
```

### Step 4 — Verify:

```bash
swapon -s
free -m
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View memory + swap | `free -m` or `free -h` |
| View swap statistics | `swapon -s` |
| Create swap on partition | `mkswap /dev/sdd2` |
| Create swap on block device | `mkswap /dev/sdc` |
| Activate all swap | `swapon -a` |
| Activate specific swap | `swapon /dev/sdd2` |
| Deactivate all swap | `swapoff -a` |
| Deactivate specific swap | `swapoff /dev/sdd2` |
| View UUID | `lsblk -fp` or `blkid /dev/sdd2` |
| Edit fstab | `vi /etc/fstab` |
| Install Cockpit storage | `dnf install -y cockpit-storaged` |

---

# Swap Priority — Example

```
/usr/lib/systemd/system/
    │
    ▼
/etc/systemd/system/
    │
    ▼
/etc/systemd/system/httpd.service.d/
└── 99-nice.conf
```

### Priority values:

| Priority | Usage | Example |
|----------|-------|---------|
| High (e.g., `10`) | Used first | Fastest swap devices |
| Same priority (e.g., `10, 10`) | Striped across devices | Better performance |
| Low (e.g., `5`) | Used after higher priorities | Slower swap devices |
| Negative (system-allocated) | Used last | Default |

---

# Swap Activation Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. System boots                                              │
│              ↓                                               │
│ 2. /etc/fstab is processed                                   │
│              ↓                                               │
│ 3. Swap units are not activated automatically by fstab     │
│              ↓                                               │
│ 4. /etc/fstab provides the configuration (UUID, priority)   │
│              ↓                                               │
│ 5. swapon -a activates swap using fstab configuration       │
│              ↓                                               │
│ 6. Swap is now active and available                         │
└─────────────────────────────────────────────────────────────┘
```

---

# Key Takeaways

- **Swap space** = disk space used as backup for RAM.
- **`free -m`** shows memory + swap usage.
- **`swapon -s`** shows active swap devices + priorities.
- **`mkswap`** creates a swap file system.
- **`swapon -a`** activates all swap from `/etc/fstab`.
- **`swapoff -a`** deactivates all swap.
- **Persistent swap** requires an entry in `/etc/fstab`.
- **Swap priority** (`pri=X`) determines which swap device is used first.
- **Same priority** = striped across devices (parallel use).
- **Higher priority** = used before lower priority.
- **Cockpit** (RHEL 10) can create swap space from the Storage panel.
- **Swap is slower than RAM** — not a substitute for sufficient memory.
- **Avoid sharing** a block device between swap and other workloads (contention).