# Managing Temporary Files — Notes

## 1. The Problem — Temporary Files Need Management

Various services and processes create files in temporary directories:

| Directory | Purpose | Cleanup |
|-----------|---------|---------|
| `/tmp/` | Temporary files (all users) | Cleaned every **10 days** (by default) |
| `/var/tmp/` | Temporary files (persistent across reboots) | Cleaned every **30 days** (by default) |
| `/run/` | Runtime data | **Deleted on every boot** |

> ⚠️ **Never store important files below `/run/`** — contents are purged on reboot.

---

## 2. The Framework — `systemd-tmpfiles`

- **`systemd-tmpfiles`** = manages creation, deletion, and cleanup of temporary files/directories
- Behavior defined by **configuration files**
- The executable belongs to **man page section 8** (system administration commands)

### Check the man page:

```bash
man 8 systemd-tmpfiles
```

---

## 3. Configuration File Locations (Precedence)

| Priority | Location | Purpose |
|----------|----------|---------|
| 1 (lowest) | `/usr/lib/tmpfiles.d/*.conf` | System-provided (RPMs) — **do not edit** |
| 2 | `/run/tmpfiles.d/*.conf` | Runtime (volatile) |
| 3 (highest) | `/etc/tmpfiles.d/*.conf` | **Customizations** (you create these) |

> 💡 **Rule of thumb:** Never edit files in `/usr/lib/` — they are owned by the operating system and will be overwritten on updates.

---

## 4. The Service and Timer Units

### Service unit — `systemd-tmpfiles-clean.service`:

```bash
systemctl --no-pager status systemd-tmpfiles-clean.service
```

**Key details:**
- `Type=oneshot` — runs once and exits
- `ExecStart=systemd-tmpfiles --clean` — the cleanup command

### Timer unit — `systemd-tmpfiles-clean.timer`:

```bash
systemctl --no-pager status systemd-tmpfiles-clean.timer
```

**Key directives:**
- `OnBootSec=15min` — runs 15 minutes after boot
- `OnUnitActiveSec=1d` — runs 1 day after last activation

> 💡 The timer ensures cleanup runs **regularly** without requiring manual intervention.

---

## 5. Example Configuration — `tmp.conf`

### Location:

```bash
cat /usr/lib/tmpfiles.d/tmp.conf
```

### Contents:

```
# Directory creation and cleanup policies
d /tmp 1777 root root 10d
d /var/tmp 1777 root root 30d
```

### Directive format:

```
type path mode user group age
```

| Field | Meaning | Example |
|-------|---------|---------|
| `d` | Create directory if missing | `d` |
| `/tmp` | Path to manage | `/tmp` |
| `1777` | Permissions (sticky bit) | `1777` |
| `root` | Owner | `root` |
| `root` | Group | `root` |
| `10d` | Cleanup age (10 days) | `10d` |

---

## 6. Common `systemd-tmpfiles` Directives

| Directive | Meaning |
|-----------|---------|
| `d` | Create directory if missing, clean files |
| `D` | Create directory, clean files, remove directory if empty |
| `f` | Create file (if missing) |
| `F` | Create file (overwrite if exists) |
| `L` | Create symlink |
| `c` | Create character device |
| `b` | Create block device |
| `e` | Clean up files in directory |
| `x` | Exclude from cleanup |
| `X` | Exclude directory from cleanup (but clean contents) |

---

## 7. Configuration Precedence Summary

```
/usr/lib/tmpfiles.d/*.conf   ← System (lowest priority)
        ↓
/run/tmpfiles.d/*.conf       ← Runtime (medium priority)
        ↓
/etc/tmpfiles.d/*.conf       ← Custom (highest priority)
```

> 💡 In case of conflicts, the **highest priority** location wins.

---

## 8. Manually Invoking `systemd-tmpfiles`

### Create directories (from config files):

```bash
systemd-tmpfiles --create
```

### Clean up old files (from config files):

```bash
systemd-tmpfiles --clean
```

### Remove directories (from config files):

```bash
systemd-tmpfiles --remove
```

### View options:

```bash
man 8 systemd-tmpfiles
```

---

## 9. Creating Custom Temporary File Management

### Step 1 — Create a custom configuration file:

```bash
sudo vi /etc/tmpfiles.d/mycustom.conf
```

### Step 2 — Add directives:

```
# Create /tmp/mycache with permissions 755, owned by user:group
d /tmp/mycache 0755 student student 7d

# Clean /var/log/temp but don't delete the directory
e /var/log/temp - - - 14d

# Exclude /tmp/special from cleanup
x /tmp/special/*
```

### Step 3 — Apply changes:

```bash
sudo systemd-tmpfiles --create
```

### Step 4 — Test cleanup:

```bash
sudo systemd-tmpfiles --clean
```

---

## 10. Age Format — Understanding Cleanup Policies

| Format | Meaning | Example |
|--------|---------|---------|
| `10d` | 10 days | Files accessed >10 days ago are deleted |
| `30m` | 30 minutes | For testing |
| `1h` | 1 hour | |
| `2w` | 2 weeks | |
| `1M` | 1 month (30 days) | |
| `1y` | 1 year | |

> 📌 The age is measured from the **last access time** (`atime`) of the file.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View tmpfiles service | `systemctl --no-pager status systemd-tmpfiles-clean.service` |
| View tmpfiles timer | `systemctl --no-pager status systemd-tmpfiles-clean.timer` |
| View system tmpfiles config | `cat /usr/lib/tmpfiles.d/tmp.conf` |
| View custom tmpfiles config | `ls -l /etc/tmpfiles.d/` |
| Create directories from config | `systemd-tmpfiles --create` |
| Clean old files | `systemd-tmpfiles --clean` |
| Remove directories | `systemd-tmpfiles --remove` |
| View tmpfiles man page | `man 8 systemd-tmpfiles` |
| View tmpfiles.d man page | `man 5 tmpfiles.d` |

---

# Temporary Directories — Comparison

| Directory | Cleanup Policy | Purpose |
|-----------|----------------|---------|
| `/tmp/` | 10 days (default) | General temporary files |
| `/var/tmp/` | 30 days (default) | Temporary files that survive reboots |
| `/run/` | Purged on reboot | Runtime data (PIDs, sockets, etc.) |

---

# Configuration Precedence Example

```
/usr/lib/tmpfiles.d/tmp.conf:
   d /tmp 1777 root root 10d

/run/tmpfiles.d/override.conf:
   d /tmp 1777 root root 5d

/etc/tmpfiles.d/myoverride.conf:
   d /tmp 1777 root root 3d

Result: /tmp cleaned every 3 days (highest precedence wins)
```

---

# Key Takeaways

- **`systemd-tmpfiles`** manages creation, cleanup, and deletion of temporary files.
- **Configuration files** define the behavior (`/usr/lib/` = system, `/etc/` = custom).
- **Precedence:** `/etc/tmpfiles.d/` > `/run/tmpfiles.d/` > `/usr/lib/tmpfiles.d/`.
- **Never edit** `/usr/lib/tmpfiles.d/` files — they are owned by RPMs.
- **Default cleanup policies:** `/tmp/` = 10 days, `/var/tmp/` = 30 days.
- **`/run/`** is purged on every boot — never store important files there.
- The **timer unit** (`systemd-tmpfiles-clean.timer`) runs the cleanup automatically:
  - 15 minutes after boot
  - Every 1 day after last activation
- Use **`systemd-tmpfiles --create`** to apply configuration changes.
- Use **`systemd-tmpfiles --clean`** to manually clean old files.
- **`man 5 tmpfiles.d`** shows all configuration directives.