# Interpreting and Managing Syslog Events — Notes

## 1. The rsyslog Service

- Managed by systemd: `rsyslog.service`
- Sends syslog messages to files below `/var/log/`
- Behavior defined by **rules** in configuration files

### Configuration sources:

| Location | Purpose |
|----------|---------|
| `/etc/rsyslog.conf` | Main configuration (owned by rsyslog package) — **avoid editing** |
| `/etc/rsyslog.d/*.conf` | **Drop-in files** for customizations (recommended) |

---

## 2. Syslog Protocol — Facilities and Priorities

- Syslog is a **protocol** (like HTTP, SMTP)
- Each message has a **facility** and a **priority**

### Facilities — Subsystems generating messages:

| Facility | Purpose |
|----------|---------|
| `auth`, `authpriv` | Security-related (sudo, su, SSH) |
| `cron` | Cron/Anacron activity |
| `daemon` | General daemons |
| `kern` | Kernel messages |
| `lpr` | Printing |
| `mail` | Mail programs (Postfix, Sendmail, etc.) |
| `news` | News servers |
| `syslog` | Internal syslog messages |
| `user` | User-generated messages |
| `local0` – `local7` | **User-definable** (for custom applications) |

> 💡 Use `local0` through `local7` for your own custom logging needs.

### Priorities — Urgency of messages (ascending order):

| Priority | Meaning | Includes |
|----------|---------|----------|
| `debug` | Very verbose, debugging info | Everything |
| `info` | Informational | info, notice, warning, err, crit, alert, emerg |
| `notice` | Normal but significant | notice, warning, err, crit, alert, emerg |
| `warning` (or `warn`) | Warning conditions | warning, err, crit, alert, emerg |
| `err` (or `error`) | Error conditions | err, crit, alert, emerg |
| `crit` | Critical conditions | crit, alert, emerg |
| `alert` | Action must be taken immediately | alert, emerg |
| `emerg` (or `panic`) | System is unusable | emerg |

> ⚠️ `error`, `warn`, and `panic` are **deprecated** — use `err`, `warning`, and `emerg`.

### Rule syntax:

```
facility.priority    /path/to/log/file
```

### Examples:

```
local5.*    /var/log/sshd.log
authpriv.*  /var/log/secure
mail.*      -/var/log/maillog
```

> 💡 A dash (`-`) before the path (e.g., `-/var/log/maillog`) means **asynchronous** writing — prevents thrashing the filesystem.

---

## 3. Common rsyslog Rules

| Rule | Destination | Content |
|------|-------------|---------|
| `*.info;mail.none;authpriv.none;cron.none` | `/var/log/messages` | General messages (excludes mail, auth, cron) |
| `authpriv.*` | `/var/log/secure` | Security events |
| `mail.*` | `-/var/log/maillog` | Mail events (async) |
| `cron.*` | `/var/log/cron` | Cron events |
| `local7.*` | `/var/log/boot.log` | Boot messages |
| `uucp,news.crit` | `/var/log/spooler` | Legacy |

> ⚠️ **Not everything goes to `/var/log/messages`.** Security logs → `secure`, Cron logs → `cron`, Mail logs → `maillog`.

---

## 4. Configuration Precedence — Lexicographical Order

- Files in `/etc/rsyslog.d/` are processed in **lexicographical order**.
- Files beginning with smaller numbers are processed first.

### Example:

```
00-logging.conf   ← Processed first
50-redhat.conf    ← Processed later (overrides previous)
80-logging.conf   ← Processed later (overrides previous)
```

> 💡 **Naming convention:** `NN-name.conf` (two-digit number, dash, name, `.conf`).

---

## 5. Changing SSH Syslog Facility — Example

### Step 1 — Check current SSH syslog facility:

```bash
grep -r SyslogFacility /etc/ssh/sshd_config.d/*.conf
```

**Output:** `50-redhat.conf:SyslogFacility AUTHPRIV`

### Step 2 — Create a drop-in file with higher precedence:

```bash
vi /etc/ssh/sshd_config.d/80-logging.conf
```

**Content:**
```
SyslogFacility local5
```

### Step 3 — Reload SSH:

```bash
systemctl reload-or-restart sshd
```

### Step 4 — Create rsyslog rule for local5:

```bash
vi /etc/rsyslog.d/80-logging.conf
```

**Content:**
```
local5.*    /var/log/sshd.log
```

### Step 5 — Restart rsyslog:

```bash
systemctl reload-or-restart rsyslog
```

### Step 6 — Test with `logger`:

```bash
logger -p local5.notice "I am a notice message"
```

### Step 7 — Verify:

```bash
cat /var/log/sshd.log
```

**Output:**
```
Sep  1 14:30:15 servera student: I am a notice message
```

---

## 6. The `logger` Command

### Purpose:

- Generate custom syslog messages
- Useful for testing rsyslog configuration

### Syntax:

```bash
logger -p facility.priority "message"
```

### Examples:

```bash
logger -p local5.notice "Test message"
logger -p authpriv.info "User logged in"
logger "Default message"      # Uses user.notice
```

---

## 7. Log Rotation — Logrotate

### Why rotate logs?

- Log files grow over time
- Need to manage size and keep only recent entries

### Configuration:

| Location | Purpose |
|----------|---------|
| `/etc/logrotate.conf` | Main configuration |
| `/etc/logrotate.d/*` | Drop-in files for specific logs |

### Default rotation settings (`/etc/logrotate.conf`):

```
weekly          # Rotate weekly
rotate 4        # Keep 4 weeks of backlogs
create          # Create new empty log file after rotation
dateext         # Use date as suffix
include /etc/logrotate.d
```

### Example — rotate `sshd.log`:

```bash
vi /etc/logrotate.d/sshd
```

**Content:**
```
/var/log/sshd.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
    create 0640 root root
}
```

| Directive | Meaning |
|-----------|---------|
| `daily` | Rotate daily |
| `rotate 7` | Keep 7 days of backlogs |
| `compress` | Compress rotated logs with gzip |
| `missingok` | Don't error if log file is missing |
| `notifempty` | Don't rotate empty files |
| `create` | Create new log file after rotation |

### View logrotate man page:

```bash
man logrotate.conf
```

---

## 8. Viewing Rotated Log Files

### Original format:

```
/var/log/messages
```

### Rotated formats:

```
/var/log/messages-20250825
/var/log/messages-20250830
```

> 💡 Use `dateext` adds the date as a suffix to rotated files.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View rsyslog config | `cat /etc/rsyslog.conf` |
| View rsyslog drop-ins | `ls -l /etc/rsyslog.d/` |
| View logrotate config | `cat /etc/logrotate.conf` |
| View logrotate drop-ins | `ls -l /etc/logrotate.d/` |
| Reload rsyslog | `systemctl reload-or-restart rsyslog` |
| Generate test log | `logger -p local5.notice "Test message"` |
| View rsyslog man page | `man rsyslog.conf` |
| View logrotate man page | `man logrotate.conf` |
| View rotated logs | `ls -l /var/log/messages*` |

---

# Syslog Facilities — Quick Reference

| Facility | Use Case | Example |
|----------|----------|---------|
| `authpriv` | Security events | SSH, sudo, su |
| `cron` | Scheduled jobs | Cron, Anacron |
| `mail` | Mail systems | Postfix, Sendmail |
| `kern` | Kernel messages | Hardware, drivers |
| `local0-7` | Custom applications | Your own services |

---

# Logrotate Directives — Common Options

| Directive | Meaning |
|-----------|---------|
| `daily/weekly/monthly` | Rotation frequency |
| `rotate N` | Keep N backlogs |
| `compress` | Compress with gzip |
| `missingok` | Don't error if file missing |
| `notifempty` | Don't rotate empty files |
| `create` | Create new file after rotation |
| `dateext` | Use date as suffix |
| `size 1M` | Rotate when file reaches 1 MB |

---

# Key Takeaways

- **rsyslog** processes logs and stores them in files below `/var/log/`.
- **Syslog** is a protocol with **facilities** (subsystems) and **priorities** (urgency).
- **Facilities:** `authpriv` (security), `cron`, `mail`, `kern`, `local0-7`, etc.
- **Priorities:** `debug`, `info`, `notice`, `warning`, `err`, `crit`, `alert`, `emerg`.
- **Rule syntax:** `facility.priority /path/to/log/file`.
- **Use drop-in files** (`/etc/rsyslog.d/`) for custom rules — avoid editing `/etc/rsyslog.conf`.
- **Lexicographical ordering** determines precedence (`00` < `50` < `80`).
- **`logger`** generates test syslog messages.
- **Logrotate** manages log file growth — rotates based on size or age.
- **`man rsyslog.conf`** and **`man logrotate.conf`** are essential references.