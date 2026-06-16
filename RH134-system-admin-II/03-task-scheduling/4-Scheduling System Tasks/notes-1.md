# Systemd Timer Units — Notes

## 1. What Are Timer Units?

- Timer units are a **systemd** feature for scheduling recurring jobs
- Alternative to **cron**
- A timer unit triggers a **service unit of the same name**

### How it works:

```
foo.timer (time specification)
     │
     ▼ When time is reached
foo.service (runs the job)
```

> 💡 Timer units are built into systemd — no separate daemon needed (unlike cron).

---

## 2. Timer Units vs Cron

| Feature | Cron | Systemd Timer Units |
|---------|------|---------------------|
| Daemon | `crond` (separate) | Built into systemd |
| Syntax | 5 fields (crontab) | `OnCalendar` directive |
| Catch-up on boot | No | Yes (with `Persistent=true`) |
| Integration | Separate | Fully integrated with systemd |
| Logging | Separate | Uses journald (same as services) |

---

## 3. Viewing Timer Units

### List all timer units:

```bash
systemctl list-units --type=timer
```

### Check status of a specific timer:

```bash
systemctl --no-pager status document-backup.timer
```

> 💡 `--no-pager` prevents output from opening in `less`.

---

## 4. Timer Unit File Structure

### Location:

| Directory | Purpose |
|-----------|---------|
| `/usr/lib/systemd/system/` | System-provided (RPMs) — **do not edit** |
| `/etc/systemd/system/` | Local customizations |

### Example — `document-backup.timer`:

```ini
[Unit]
Description=Run document backup daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

### Example — `document-backup.service`:

```ini
[Unit]
Description=Backup all user Documents directories

[Service]
Type=oneshot
ExecStart=/usr/local/bin/document-backup.sh
```

---

## 5. Timer Unit — Sections and Directives

### `[Unit]` Section:

| Directive | Purpose |
|-----------|---------|
| `Description` | Human-readable description |
| `After` | Start after specified unit |

### `[Timer]` Section:

| Directive | Purpose | Example |
|-----------|---------|---------|
| `OnCalendar` | Time specification | `daily`, `weekly`, `*:0/15` |
| `OnBootSec` | After boot | `15min` |
| `OnUnitActiveSec` | After last activation | `1h` |
| `Persistent` | Catch up missed runs | `true` or `false` |
| `Unit` | Service to run (if different name) | `backup.service` |

### `[Install]` Section:

| Directive | Purpose |
|-----------|---------|
| `WantedBy` | Target that triggers this unit | `timers.target` |

> 💡 If `Unit` is not specified, it assumes the **same name** with `.service` extension.

---

## 6. The Service Unit — `Type=oneshot`

### Why `Type=oneshot`?

- Service runs, does its job, then **exits**
- Perfect for backup tasks, maintenance scripts

```ini
[Service]
Type=oneshot
ExecStart=/usr/local/bin/document-backup.sh
```

### Other `Type` options:

| Type | Behavior |
|------|----------|
| `simple` | Default — runs in foreground |
| `oneshot` | Runs once and exits |
| `forking` | Forks into background |
| `idle` | Runs when system is idle |

---

## 7. Common `OnCalendar` Time Specifications

| Format | Meaning |
|--------|---------|
| `daily` | Every day at midnight |
| `weekly` | Every week on Monday at midnight |
| `monthly` | Every month on the 1st at midnight |
| `*:0/15` | Every 15 minutes |
| `*-*-* 02:00:00` | Every day at 2:00 AM |
| `Mon..Fri 09:00:00` | Weekdays at 9:00 AM |
| `2025-12-31 23:59:00` | Specific date/time |

### Examples:

```ini
OnCalendar=daily
OnCalendar=weekly
OnCalendar=*:0/15
OnCalendar=Mon..Fri *-*-* 09:00:00
```

---

## 8. Starting and Enabling Timer Units

### Start now:

```bash
systemctl start document-backup.timer
```

### Enable at boot:

```bash
systemctl enable document-backup.timer
```

### Start + enable together:

```bash
systemctl enable --now document-backup.timer
```

### Check status:

```bash
systemctl --no-pager status document-backup.timer
```

**Expected output:**
```
Loaded: loaded (/etc/systemd/system/document-backup.timer; enabled)
Active: active (waiting)
```

---

## 9. The Backup Script Example

### Location:

```bash
/usr/local/bin/document-backup.sh
```

### Must be executable:

```bash
chmod +x /usr/local/bin/document-backup.sh
```

### Script concept:

```bash
#!/bin/bash
BACKUP_DIR=/backups
mkdir -p "$BACKUP_DIR"

# Find all users with a Documents directory
for user_home in /home/*; do
    user=$(basename "$user_home")
    if [ -d "$user_home/Documents" ]; then
        tar cf "$BACKUP_DIR/${user}_documents.tar" "$user_home/Documents"
    fi
done
```

---

## 10. Putting It All Together

### Step 1 — Create the service unit file:

```bash
sudo vi /etc/systemd/system/document-backup.service
```

```ini
[Unit]
Description=Backup all user Documents directories

[Service]
Type=oneshot
ExecStart=/usr/local/bin/document-backup.sh
```

### Step 2 — Create the timer unit file:

```bash
sudo vi /etc/systemd/system/document-backup.timer
```

```ini
[Unit]
Description=Run document backup daily at midnight

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

### Step 3 — Create the backup script:

```bash
sudo vi /usr/local/bin/document-backup.sh
sudo chmod +x /usr/local/bin/document-backup.sh
```

### Step 4 — Enable and start the timer:

```bash
sudo systemctl enable --now document-backup.timer
```

### Step 5 — Verify:

```bash
sudo systemctl --no-pager status document-backup.timer
sudo systemctl list-units --type=timer
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| List timer units | `systemctl list-units --type=timer` |
| Check timer status | `systemctl --no-pager status timer-name.timer` |
| Start timer | `systemctl start timer-name.timer` |
| Enable timer | `systemctl enable timer-name.timer` |
| Start + enable | `systemctl enable --now timer-name.timer` |
| View timer unit file | `systemctl cat timer-name.timer` |
| View service unit file | `systemctl cat service-name.service` |
| Reload systemd units | `systemctl daemon-reload` |

---

# Timer Unit File Locations

```
/usr/lib/systemd/system/
├── timer-name.timer      # System-provided (do not edit)
└── timer-name.service    # System-provided (do not edit)

/etc/systemd/system/
├── timer-name.timer      # Custom (you create these)
└── timer-name.service    # Custom (you create these)
```

> ⚠️ **Never edit files in `/usr/lib/systemd/system/`** — they are owned by RPM packages and will be overwritten on updates.

---

# Unit File Relationship Diagram

```
/usr/local/bin/document-backup.sh
         │
         │ ExecStart
         ▼
document-backup.service
[Service]
Type=oneshot
ExecStart=/usr/local/bin/document-backup.sh
         │
         │ Called by timer
         ▼
document-backup.timer
[Timer]
OnCalendar=daily
Persistent=true
[Install]
WantedBy=timers.target
         │
         │ Part of
         ▼
timers.target
         │
         │ Called by
         ▼
multi-user.target
```

---

# Key Takeaways

- **Timer units** = systemd's built-in scheduling mechanism (alternative to cron).
- **Naming convention:** `foo.timer` triggers `foo.service` (same base name).
- **Service units** should use `Type=oneshot` for one-time jobs.
- **Timer units** use `OnCalendar` for time specification (`daily`, `weekly`, `*:0/15`, etc.).
- **`Persistent=true`** catches up on missed runs (when system was offline).
- **Custom units** go in `/etc/systemd/system/` — **never edit** `/usr/lib/systemd/system/`.
- **Enable + start** timers with `systemctl enable --now timer-name.timer`.
- **`timers.target`** is the target that starts all active timer units.
- **`systemctl list-units --type=timer`** shows all timer units.
- **`systemctl cat timer-name.timer`** shows the unit file content.
- Timer units are integrated with **systemd logging** (journald) — easier troubleshooting.