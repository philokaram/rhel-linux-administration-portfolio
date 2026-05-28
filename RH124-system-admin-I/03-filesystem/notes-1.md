# Linux File System Hierarchy — Notes

## 1. The Big Picture

- Linux file system = **tree structure**
- Everything begins at **root** = `/` (forward slash)
- From the trunk, branches spread to different directories
- Each directory has a **specific purpose**

> Think of it like a tree. Learn which branch to climb, and you'll always find what you're looking for.

---

## 2. Top-Level Directories Overview

```
/
├── home/
├── etc/
├── var/
├── usr/
├── tmp/
├── var/tmp/
├── root/
├── boot/
├── mnt/
├── run/
├── dev/
├── proc/
└── sys/
```

---

## 3. User & Personal Directories

| Directory | Purpose | Example |
|-----------|---------|---------|
| `/home/` | Personal files for regular users | `/home/rgdacosta`, `/home/student` |
| `/root/` | Root user's personal home directory | (not for normal users) |

> ⚠️ Don't confuse `/root` (home directory) with `/` (root of file system).

---

## 4. Configuration Directory

| Directory | Purpose | Tip |
|-----------|---------|-----|
| `/etc/` | System configuration files | **Always back up before editing** |

> Want to change how Linux behaves? Odds are you'll edit something in `/etc`.

---

## 5. Variable Data Directory

| Directory | Purpose | Key Subdirectory |
|-----------|---------|------------------|
| `/var/` | Data that changes over time | `/var/log/` (system logs) |

**Includes:** logs, mail, spool files, temporary files

---

## 6. Software Directories (`/usr/`)

| Location | Purpose |
|----------|---------|
| `/usr/bin/` | Normal executables (binaries) for all users |
| `/usr/sbin/` | System binaries (typically for root) |
| `/usr/lib64/` | Libraries |
| `/usr/share/` | Shared data (docs, man pages, etc.) |

> Think of `/usr/` as your **software warehouse**.

---

## 7. Temporary Directories — Automatic Cleanup

| Directory | Cleanup Rule | Best Used For |
|-----------|--------------|----------------|
| `/tmp/` | Files untouched for **10 days** are deleted | Short-lived scratch space |
| `/var/tmp/` | Files untouched for **30 days** are deleted | Longer-lived temporary data |

> ⚠️ Don't leave anything important in either directory.

---

## 8. System & Boot Directories

| Directory | Purpose |
|-----------|---------|
| `/boot/` | Linux kernel + files needed to start the system |
| `/mnt/` | Traditional mount point for extra storage (USB, test file systems) |
| `/run/` | Runtime info (PIDs, sockets). Created fresh at each boot — wiped clean on restart |

> Unless you're fixing a serious issue, you won't need to look inside `/boot`.

---

## 9. Special Directories — The "Magic" of Linux

These make Linux tick. Everything is represented as **files**.

| Directory | Analogy | Contents |
|-----------|---------|----------|
| `/dev/` | Electrical panel of your system | Devices: hard drives, USB sticks, terminals |
| `/proc/` | Window into the kernel's state | Processes, CPU info, memory, running system info |
| `/sys/` | Live catalog of your hardware | Devices, drivers, kernel objects |

> In Linux, **everything is a file** — these directories prove it.

---

# Complete Directory Reference Table

| Directory | Purpose | Typical User Access |
|-----------|---------|---------------------|
| `/` | Root of the entire file system | All |
| `/home/` | User personal files | Regular users |
| `/etc/` | System configuration | Admin (root) |
| `/var/` | Variable data (logs, mail) | Admin |
| `/usr/` | Software (binaries, libraries) | All |
| `/tmp/` | Temporary files (10-day cleanup) | All |
| `/var/tmp/` | Temporary files (30-day cleanup) | All |
| `/root/` | Root user's home | Root only |
| `/boot/` | Kernel and boot files | Admin |
| `/mnt/` | Mount point for storage | Admin |
| `/run/` | Runtime data (volatile) | System |
| `/dev/` | Device files | System |
| `/proc/` | Kernel and process info | All (read-only) |
| `/sys/` | Hardware and driver info | System |

---

# Quick Navigation Tips

| If you want to... | Go to... |
|-------------------|----------|
| Save a personal document | `/home/yourusername/` |
| Change a system setting | `/etc/` |
| Check system logs | `/var/log/` |
| Find where an app is installed | `/usr/bin/` or `/usr/sbin/` |
| Use temporary scratch space | `/tmp/` |
| See what devices are connected | `/dev/` or `/sys/` |
| Check CPU/memory info | `/proc/cpuinfo`, `/proc/meminfo` |

---

# Key Takeaways

- **Everything begins at `/`** — the root of the tree.
- Linux is **not a maze** — each directory has a specific, logical purpose.
- `/home/` = your stuff. `/etc/` = system settings. `/var/` = logs.
- `/usr/` = software warehouse. `/tmp/` = scratch pad (auto-cleaned).
- Special directories (`/dev/`, `/proc/`, `/sys/`) make Linux powerful — hardware and kernel internals appear as **files**.
- Don't confuse `/root` (root's home) with `/` (file system root).
- **Always back up config files** before editing in `/etc/`.