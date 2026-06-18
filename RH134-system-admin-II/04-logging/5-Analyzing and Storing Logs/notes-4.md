# Making the Journal Persistent — Notes

## 1. Volatile vs Persistent Journal

| Location | Persistence | Default |
|----------|-------------|---------|
| `/run/log/journal/` | **Volatile** — deleted on reboot | ✅ Yes |
| `/var/log/journal/` | **Persistent** — survives reboots | ❌ No (unless configured) |

> 💡 By default, the journal is **volatile** — logs are lost on reboot.

---

## 2. Making the Journal Persistent — Two Simple Steps

### Step 1 — Create the persistent directory:

```bash
mkdir -p /var/log/journal
```

### Step 2 — Flush current journal to persistent storage:

```bash
journalctl --flush
```

### Result:

- Journal data is now stored in `/var/log/journal/`
- Survives system reboots

### Verification:

```bash
ls -l /var/log/journal/
```

**Output:**
```
drwxr-sr-x 2 root systemd-journal 4096 Sep  1 14:30 1234567890abcdef1234567890abcdef/
```

Inside:

```bash
ls -l /var/log/journal/1234567890abcdef1234567890abcdef/
```

**Files:**
```
-rw-r----- 1 root systemd-journal 8.0M Sep  1 14:30 system.journal
-rw-r----- 1 root systemd-journal 1.0M Sep  1 14:30 user.journal
```

---

## 3. Journal Configuration — `journald.conf`

### Configuration sources:

| Location | Purpose |
|----------|---------|
| `/usr/lib/systemd/journald.conf` | System default — **do not edit** |
| `/etc/systemd/journald.conf` | Main configuration — **for custom settings** |
| `/etc/systemd/journald.conf.d/*.conf` | **Drop-in files** — recommended approach |

> 💡 **Best practice:** Use drop-in files in `/etc/systemd/journald.conf.d/` rather than editing `journald.conf` directly.

### Man page:

```bash
man journald.conf
```

---

## 4. Important Configuration Directives

| Directive | Purpose | Example |
|-----------|---------|---------|
| `Storage` | Controls where logs are stored | `persistent`, `volatile`, `auto`, `none` |
| `SystemMaxUse` | Maximum total journal size | `1G` |
| `SystemMaxFileSize` | Maximum individual journal file size | `200M` |
| `MaxRetentionSec` | Maximum age of journal files | `7d` |
| `Compress` | Compress journal files | `yes` |
| `ForwardToSyslog` | Forward to rsyslog | `yes` |
| `ForwardToConsole` | Forward to console | `no` |
| `ForwardToWall` | Forward to all logged-in users | `no` |
| `Audit` | Include audit messages | `yes` |
| `RateLimitIntervalSec` | Rate limiting window | `30s` |
| `RateLimitBurst` | Max messages in window | `1000` |

### Example — persistent journal with limits:

```bash
vi /etc/systemd/journald.conf.d/99-custom.conf
```

**Content:**
```ini
[Journal]
Storage=persistent
SystemMaxUse=1G
SystemMaxFileSize=200M
MaxRetentionSec=7d
Compress=yes
ForwardToSyslog=yes
ForwardToConsole=no
Audit=yes
RateLimitIntervalSec=30s
RateLimitBurst=1000
```

---

## 5. Storage Directive — Options

| Value | Behavior |
|-------|----------|
| `volatile` | Store in `/run/log/journal/` only (volatile) |
| `persistent` | Store in `/var/log/journal/` (persistent) |
| `auto` | Persistent if `/var/log/journal/` exists, otherwise volatile |
| `none` | Disable logging (not recommended) |

> 💡 Default is `auto` — the journal becomes persistent if the directory exists.

---

## 6. Applying Configuration Changes

### Step 1 — Create/modify configuration:

```bash
vi /etc/systemd/journald.conf.d/99-custom.conf
```

### Step 2 — Restart journald:

```bash
systemctl restart systemd-journald
```

### Step 3 — Flush existing logs to persistent storage:

```bash
journalctl --flush
```

### Step 4 — Verify:

```bash
ls -l /var/log/journal/
```

---

## 7. Benefits of Persistent Journal

### Access logs from previous boots:

```bash
journalctl -b -1
```

### List all available boots:

```bash
journalctl --list-boots
```

**Output:**
```
-2 1234567890abcdef1234567890abcdef Tue 2025-08-30 08:00:00 UTC—Tue 2025-08-30 14:00:00 UTC
-1 2345678901abcdef2345678901abcdef Tue 2025-08-31 08:00:00 UTC—Tue 2025-08-31 14:00:00 UTC
 0 3456789012abcdef3456789012abcdef Tue 2025-09-01 08:00:00 UTC—Tue 2025-09-01 14:00:00 UTC
```

> 💡 With persistent journal, you can troubleshoot issues from **previous boots**.

---

## 8. File Naming in `/var/log/journal/`

### Directory structure:

```
/var/log/journal/
└── 1234567890abcdef1234567890abcdef/   # System ID (unique)
    ├── system.journal                    # System messages
    └── user.journal                      # User messages
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Create persistent journal directory | `mkdir -p /var/log/journal` |
| Flush journal to persistent storage | `journalctl --flush` |
| View journal config man page | `man journald.conf` |
| View journal config | `cat /etc/systemd/journald.conf` |
| View custom drop-in config | `cat /etc/systemd/journald.conf.d/*.conf` |
| Restart journald | `systemctl restart systemd-journald` |
| List available boots | `journalctl --list-boots` |
| View previous boot logs | `journalctl -b -1` |

---

# Configuration Summary — Volatile vs Persistent

| Setting | Storage Location | After Reboot |
|---------|-------------------|--------------|
| Default (Storage=auto) | `/run/log/journal/` (if no persistent dir) | Lost |
| Persistent (Storage=persistent) | `/var/log/journal/` | Survives |
| After `mkdir /var/log/journal` | `/var/log/journal/` | Survives |
| After `journalctl --flush` | `/var/log/journal/` (copied from /run) | Survives |

---

# Key Takeaways

- **Volatile journal** is default — stored in `/run/log/journal/` (lost on reboot).
- **Persistent journal** is stored in `/var/log/journal/` (survives reboots).
- **Two simple steps to make it persistent:**
  1. `mkdir -p /var/log/journal`
  2. `journalctl --flush`
- **Configuration** is controlled by `journald.conf` (system default) and drop-in files in `/etc/systemd/journald.conf.d/`.
- **Storage directive options:** `volatile`, `persistent`, `auto` (default), `none`.
- **Use drop-in files** (`/etc/systemd/journald.conf.d/`) for custom configurations.
- **Restart** `systemd-journald` after configuration changes: `systemctl restart systemd-journald`.
- **Persistent journal** enables:
  - `journalctl -b -1` (logs from previous boot)
  - `journalctl --list-boots` (list all available boots)
- **Rate limiting** (`RateLimitIntervalSec` + `RateLimitBurst`) prevents log flooding.