# Mounting NFS Shares — Notes

## 1. What Is NFS?

- **NFS** = Network File System
- Decades-old standard for sharing files across Linux and Unix systems
- A central server makes directories available → clients mount them as if they were local

### NFS Versions:

| Version | Transport | Approach |
|---------|-----------|----------|
| **NFSv3** | TCP or UDP | Each directory is a separate export; uses RPC bind |
| **NFSv4** (RHEL 10 default) | TCP (more reliable) | Single **export tree** (virtual root) with all shared directories |

> 💡 RHEL 10 primarily uses **NFS version 4.2**.

---

## 2. NFS Server Setup (for Reference)

### NFS server service:

```bash
systemctl status nfs-server
```

### Export configuration:

```bash
cat /etc/exports.d/isos.exports
```

**Example:**
```
/isos *(rw)
```

| Part | Meaning |
|------|---------|
| `/isos` | Directory being shared |
| `*` | Share with everyone |
| `(rw)` | **Read-write** access |

---

## 3. Discovering NFS Exports — NFSv4 vs NFSv3

### NFSv3 — `showmount`:

```bash
showmount -e serverb
```

> ⚠️ This requires RPC bind service on the server (not available with NFSv4-only setup) → may time out.

### NFSv4 — Mount the export root:

```bash
mount serverb:/ /mnt
ls /mnt
```

**Result:** Shows all exported directories (e.g., `/isos`)

### Unmount:

```bash
umount /mnt
```

---

## 4. Mounting NFS Shares — Three Methods

| Method | Command | Persistence | Pros | Cons |
|--------|---------|-------------|------|------|
| **Manual** | `mount` | Lost on reboot | Quick, simple | Not persistent |
| **Persistent** | `/etc/fstab` | Survives reboots | Always available | Can stall boot if server is down |
| **On-demand** | AutoFS | Survives reboots | No boot delays | Covered in next section |

---

## 5. Manual Mounting — Temporary Access

```bash
mkdir -p /mnt/isos
mount serverb:/isos /mnt/isos
```

### Verify:

```bash
ls -lh /mnt/isos
```

### Unmount:

```bash
umount /mnt/isos
```

> 💡 Use manual mounting for quick, one-time access.

---

## 6. Persistent Mounting — `/etc/fstab`

### Add entry:

```
serverb:/isos /mnt/isos nfs4 defaults 0 0
```

| Field | Value | Meaning |
|-------|-------|---------|
| 1 | `serverb:/isos` | NFS export (server:directory) |
| 2 | `/mnt/isos` | Local mount point |
| 3 | `nfs4` | File system type (NFS version 4) |
| 4 | `defaults` | Mount options |
| 5 | `0` | No dump backup |
| 6 | `0` | No fsck |

### Test without rebooting:

```bash
umount /mnt/isos
mount -av
ls /mnt/isos
```

> 💡 **Exam tip:** Always test `/etc/fstab` with `mount -av` before rebooting to avoid boot delays.

---

## 7. NFS Mount Options

| Option | Meaning |
|--------|---------|
| `rw` | Read-write access |
| `ro` | Read-only access |
| `soft` | Return error if server doesn't respond (fewer hangs) |
| `hard` | Keep retrying until server responds (default) |
| `intr` | Allow interrupts for hung NFS operations |
| `noatime` | Don't update access timestamps (better performance) |

> ⚠️ **Server has final say:** If the NFS server exports a directory as read-only (`ro`), the client's `rw` request is ignored.

---

## 8. Troubleshooting — "Device is Busy"

### Error:

```bash
umount /mnt/isos
umount: /mnt/isos: target is busy
```

### Cause: A process or terminal session is using the directory.

### Solution — Find the process:

```bash
lsof /mnt/isos
```

**Output:**
```
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash    1234 root  cwd    DIR   0,51     4096    2 /mnt/isos
```

### Fix:

1. Change out of the directory: `cd /`
2. Close any terminal sessions in that directory
3. Try `umount` again

---

## 9. NFS Architecture — NFSv4 Export Tree

```
NFS Server: serverb
Export Tree (NFSv4 root: /)
           │
           └── isos/     ← Exported directory
                 ├── rhel9.iso
                 ├── rhel10.iso
                 └── fedora.iso
```

### Client connects:

```bash
mount serverb:/isos /mnt/isos
```

Client sees:

```
/mnt/isos/
├── rhel9.iso
├── rhel10.iso
└── fedora.iso
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Check NFS server status | `systemctl status nfs-server` |
| View exports on server | `cat /etc/exports.d/*.exports` |
| Discover exports (NFSv4) | `mount serverb:/ /mnt && ls /mnt && umount /mnt` |
| Discover exports (NFSv3) | `showmount -e serverb` |
| Manual mount NFS share | `mount serverb:/isos /mnt/isos` |
| Persistent mount (fstab) | Add to `/etc/fstab`, then `mount -av` |
| Unmount NFS share | `umount /mnt/isos` |
| Find process using mount | `lsof /mnt/isos` |
| View NFS mount options | `man nfs` |

---

# NFSv3 vs NFSv4 — Comparison

| Feature | NFSv3 | NFSv4 |
|---------|-------|-------|
| Transport | TCP or UDP | TCP only |
| Discovery | `showmount -e` | Mount server root: `/` |
| Exports | Separate directories | Single export tree |
| RPC bind | Required | Not required |
| Reliability | Less reliable | More reliable |
| Firewall | Complex (multiple ports) | Simple (port 2049) |

---

# NFS Mount — Common Options

```
serverb:/isos /mnt/isos nfs4 defaults,noatime 0 0
```

| Option | Purpose |
|--------|---------|
| `defaults` | rw, suid, dev, exec, auto, nouser, async |
| `soft` | Return error on timeout (better for clients) |
| `hard` | Retry indefinitely (default) |
| `intr` | Allow interrupts |
| `noatime` | Better performance |
| `vers=4.2` | Force NFS version 4.2 |

---

# Key Takeaways

- **NFS** = Network File System for sharing files across Linux/Unix systems.
- **RHEL 10 uses NFS version 4.2** by default (TCP-based, export tree).
- **NFSv3** uses `showmount -e` for discovery; **NFSv4** mounts the server root: `mount serverb:/ /mnt`.
- **Manual mount:** Quick but lost on reboot.
- **Persistent mount:** Add to `/etc/fstab` and test with `mount -av`.
- **fstab format:** `server:/export /mountpoint nfs4 defaults 0 0`.
- **Server has final say** on read-write vs read-only access.
- **"Device is busy"** → use `lsof` to find the process using the mount point.
- **Mount options** include `rw`, `ro`, `soft`, `hard`, `intr`, `noatime`.
- **AutoFS** (next section) provides on-demand mounting — no boot delays.
- **Setting up an NFS server is beyond the scope of this course** — focus on client-side mounting.