# Auto-Mounting Storage Devices — Notes

## 1. What Is Autofs?

- **autofs** = service that automatically mounts file systems **on demand**
- Mounts when accessed → unmounts after a period of inactivity
- Prevents boot delays (unlike `/etc/fstab`)

### Benefits:

| Benefit | Description |
|---------|-------------|
| No boot delays | File systems only mount when accessed |
| Saves resources | Unmounts when idle |
| Always latest config | Not stale from last reboot |
| Non-root users | Can access remote shares without `sudo` |
| Smart failover | Can pick fastest server if multiple available |

> 💡 **Key advantage:** Normal users don't need `sudo` to access remote shares.

---

## 2. Autofs Architecture — Map Files

```
/etc/auto.master.d/*.autofs   ← Master map files (point to other maps)
            │
            ├── Points to → /etc/auto.direct   ← Direct map (fixed directories)
            │
            └── Points to → /etc/auto.indirect ← Indirect map (dynamic subdirectories)
```

### Map Types:

| Type | Description | Mount Point | Example |
|------|-------------|-------------|---------|
| **Direct** | Fixed, permanent directories | Fixed path (e.g., `/mnt/docs`) | Always available as a directory |
| **Indirect** | Dynamic subdirectories on demand | Base path + subdirectory | Directories appear only when accessed |

---

## 3. Master Map Files — `/etc/auto.master.d/*.autofs`

### Location:

```
/etc/auto.master.d/*.autofs
```

> 💡 Do not edit `/etc/auto.master` directly — use drop-in files in `auto.master.d/`.

### Direct map entry:

```bash
/- /etc/auto.direct
```

| Part | Meaning |
|------|---------|
| `/-` | Direct map marker (mount to fixed directories) |
| `/etc/auto.direct` | File defining what gets mounted where |

### Indirect map entry:

```bash
/home/student/isos /etc/auto.indirect
```

| Part | Meaning |
|------|---------|
| `/home/student/isos` | Base directory (mount point) |
| `/etc/auto.indirect` | File defining subdirectories to mount |

---

## 4. Direct Map — `auto.direct`

### Format:

```
/mountpoint -fstype=nfs4 server:/export
```

### Example:

```
/mnt/docs     -fstype=nfs4 serverb:/docs
/mnt/training -fstype=nfs4 serverb:/training
```

### What it does:

- When you access `/mnt/docs` → automount mounts `serverb:/docs`
- When you access `/mnt/training` → automount mounts `serverb:/training`

> 💡 Direct maps are for **fixed, predictable** mount points.

---

## 5. Indirect Map — `auto.indirect`

### Format:

```
subdir -fstype=nfs4 server:/export/&
```

### Example:

```
* -fstype=nfs4 serverb:/isos/&
```

| Part | Meaning |
|------|---------|
| `*` | Wildcard — matches whatever subdirectory you request |
| `&` | Replaced with the requested subdirectory name |
| `serverb:/isos/&` | NFS server export + requested subdirectory |

### How it works:

| You access | This mounts |
|------------|-------------|
| `/home/student/isos/rhel` | `serverb:/isos/rhel` |
| `/home/student/isos/fedora` | `serverb:/isos/fedora` |
| `/home/student/isos/centos_stream` | `serverb:/isos/centos_stream` |

> 💡 Indirect maps create **dynamic** subdirectories on demand.

---

## 6. Installing and Starting Autofs

### Install:

```bash
dnf install -y autofs
```

### Enable and start:

```bash
systemctl enable --now autofs
```

### Verify:

```bash
systemctl status autofs
```

---

## 7. Putting It All Together — Workflow

### Step 1 — Create master map files:

```bash
vi /etc/auto.master.d/direct.autofs
```

**Content:**
```
/- /etc/auto.direct
```

```bash
vi /etc/auto.master.d/indirect.autofs
```

**Content:**
```
/home/student/isos /etc/auto.indirect
```

### Step 2 — Create map files:

```bash
vi /etc/auto.direct
```

**Content:**
```
/mnt/docs     -fstype=nfs4 serverb:/docs
/mnt/training -fstype=nfs4 serverb:/training
```

```bash
vi /etc/auto.indirect
```

**Content:**
```
* -fstype=nfs4 serverb:/isos/&
```

### Step 3 — Start autofs:

```bash
systemctl enable --now autofs
```

---

## 8. Testing Autofs — Direct Map

### Before access:

```bash
mount | grep -i nfs
```

**Output:** (no NFS mounts)

### Access the mount point:

```bash
ls /mnt/docs
```

### After access:

```bash
mount | grep -i nfs
```

**Output:** `serverb:/docs on /mnt/docs type nfs4`

> 💡 The mount happens **when you access the directory**, not before.

---

## 9. Testing Autofs — Indirect Map

### Before access:

```bash
mount | grep isos
```

**Output:** (no `isos` mounts)

### Access a subdirectory:

```bash
ls /home/student/isos/rhel
```

### After access:

```bash
mount | grep isos
```

**Output:** `serverb:/isos/rhel on /home/student/isos/rhel type nfs4`

### Try a non-existent subdirectory:

```bash
ls /home/student/isos/foo
```

**Result:** `ls: cannot access /home/student/isos/foo: No such file or directory`

> 💡 The wildcard (`*`) only mounts what exists on the server.

---

## 10. Autofs Unmounting

- After a period of **inactivity**, autofs automatically unmounts the file system
- Default timeout is **5 minutes** (configurable)

### View timeout:

```bash
grep timeout /etc/auto.master
```

### Change timeout (in master map):

```
/- /etc/auto.direct --timeout=120
```

**Meaning:** Unmount after 120 seconds of inactivity.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Install autofs | `dnf install -y autofs` |
| Enable and start autofs | `systemctl enable --now autofs` |
| Check autofs status | `systemctl status autofs` |
| View master maps | `ls -l /etc/auto.master.d/` |
| View direct map | `cat /etc/auto.direct` |
| View indirect map | `cat /etc/auto.indirect` |
| Check mounted NFS shares | `mount \| grep -i nfs` |
| Restart autofs | `systemctl restart autofs` |
| View autofs man page | `man autofs` |

---

# Direct vs Indirect Map — Comparison

| Feature | Direct Map | Indirect Map |
|---------|------------|--------------|
| Mount point | Fixed directory | Base + dynamic subdirectory |
| Entry format | `/mountpoint server:/export` | `* server:/export/&` |
| Directory exists | Always (as directory) | Only when mounted |
| Example | `/mnt/docs` | `/home/student/isos/rhel` |
| Use case | Fixed, predictable locations | User home directories, dynamic content |

---

# Map File Syntax Summary

### Direct:

```
/mountpoint -fstype=nfs4 server:/export
```

### Indirect:

```
subdir -fstype=nfs4 server:/export/&
```

### Options (between mount point and server):

```
/mountpoint -fstype=nfs4,ro,noatime server:/export
```

---

# Key Takeaways

- **Autofs** mounts file systems **on demand** — no boot delays.
- **Master maps** (`/etc/auto.master.d/*.autofs`) point to other map files.
- **Direct maps** mount to fixed, permanent directories (`/-`).
- **Indirect maps** create dynamic subdirectories (`/base/path`).
- **Wildcard (`*`)** = any subdirectory requested.
- **Ampersand (`&`)** = the specific subdirectory requested.
- **Users don't need `sudo`** to access autofs-mounted shares.
- **NFS is common** — autofs also supports SMB/CIFS and local file systems.
- **Timeout** default is 5 minutes — configurable in master map.
- **Restart autofs** after making configuration changes:
  ```bash
  systemctl restart autofs
  ```