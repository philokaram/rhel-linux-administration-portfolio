# System Log Architecture — Notes

## 1. The Two Logging Services

| Service | Purpose | Storage Location |
|---------|---------|------------------|
| `systemd-journald` | Collects logs from all sources | `/run/log/journal/` (indexed database, **volatile**) |
| `rsyslog` | Processes logs and stores in files | `/var/log/` (plain text files, **persistent**) |

> 💡 `systemd-journald` is built into systemd. `rsyslog` works **with** it — they complement each other.

---

## 2. `systemd-journald` — The Collector

- Part of systemd (`systemd-journald.service`)
- Collects logs from **multiple sources**:

| Source | Examples |
|--------|----------|
| Kernel | Kernel messages |
| Boot output | Startup messages |
| Services (daemons) | Standard output and error |
| Syslog protocol | Log messages from applications |

### Storage:

```
/run/log/journal/
```

- Indexed database (not plain text files)
- **Volatile** — contents deleted on reboot (recreated by tmpfiles framework)

### Interface — `journalctl`:

```bash
journalctl
```

---

## 3. `rsyslog` — The Processor

- Works with `systemd-journald` (not a separate framework)
- Takes syslog events from `systemd-journald`
- Stores them in **plain text files** below `/var/log/`

### Configuration:

```bash
/etc/rsyslog.conf
```

### Common log files:

| Log File | Contains |
|----------|----------|
| `/var/log/messages` | General system messages (not everything!) |
| `/var/log/secure` | Security-related events (sudo, su, SSH) |
| `/var/log/boot.log` | Boot output |
| `/var/log/maillog` | Mail-related events |
| `/var/log/cron` | Cron job activity |

> ⚠️ **Not everything goes into `/var/log/messages`.** Security logs go to `/var/log/secure`, etc.

---

## 4. Viewing Logs — `tail -f`

### View live updates:

```bash
tail -f /var/log/messages
```

| Option | Meaning |
|--------|---------|
| `-f` | **F**ollow — show new entries as they are added |

### Quit:

Press **`Ctrl+C`**

### View last N lines:

```bash
tail -n 20 /var/log/messages
```

### View entire file:

```bash
cat /var/log/messages
```

> 💡 For large log files, use `less` instead of `cat`: `less /var/log/messages`

---

## 5. Verifying the Services

### Check `systemd-journald`:

```bash
systemctl --no-pager status systemd-journald
```

### Check `rsyslog`:

```bash
systemctl --no-pager status rsyslog
```

---

## 6. Log Flow Summary

```
Kernel Messages
      │
Boot Output ──────┐
      │            │
Service Output ────┼──▶ systemd-journald ──▶ /run/log/journal/ (volatile database)
      │            │
Syslog Protocol ───┘              │
                                  │
                                  ▼
                              rsyslog ──▶ /var/log/*.log (persistent files)
```

---

## 7. Volatile vs Persistent Storage

| Location | Purpose | Persistence |
|----------|---------|-------------|
| `/run/log/journal/` | Journald database | **Volatile** — lost on reboot |
| `/var/log/` | Rsyslog log files | **Persistent** — survives reboots |

> 💡 `rsyslog` ensures logs are stored persistently even if the system reboots.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Check journald status | `systemctl --no-pager status systemd-journald` |
| Check rsyslog status | `systemctl --no-pager status rsyslog` |
| View journald logs | `journalctl` |
| View all system logs | `cat /var/log/messages` |
| View last 20 lines | `tail -n 20 /var/log/messages` |
| Follow log live | `tail -f /var/log/messages` |
| View boot log | `cat /var/log/boot.log` |
| View security log | `cat /var/log/secure` |
| View cron log | `cat /var/log/cron` |

---

# System Log File Locations

```
/var/log/
├── messages      # General system messages
├── secure        # Security/auth events (sudo, su, SSH)
├── boot.log      # Boot process output
├── maillog       # Mail events
├── cron          # Cron job activity
├── httpd/        # HTTP server logs
├── audit/        # Audit logs (SELinux, etc.)
└── ...
```

---

# Key Takeaways

- **`systemd-journald`** = collects all logs from the system.
- **`rsyslog`** = processes logs and stores them in persistent files.
- **Journald logs** are stored in `/run/log/journal/` (volatile, indexed database).
- **Rsyslog logs** are stored in `/var/log/` (persistent, plain text files).
- Not everything goes to `/var/log/messages`:
  - Security → `/var/log/secure`
  - Boot → `/var/log/boot.log`
  - Mail → `/var/log/maillog`
  - Cron → `/var/log/cron`
- **`tail -f`** shows live updates to log files (quit with `Ctrl+C`).
- **`journalctl`** queries the journald database.
- **Use `less`** instead of `cat` for large log files.