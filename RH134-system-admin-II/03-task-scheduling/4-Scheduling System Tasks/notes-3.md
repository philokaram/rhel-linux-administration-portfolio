# Scheduling Recurring System Tasks with Cron — Notes

## 1. User Cron vs System Cron

### User cron (covered previously):

```
* * * * * command
│ │ │ │ │
│ │ │ │ └─── Day of week
│ │ │ └───── Month
│ │ └─────── Day of month
│ └───────── Hour
└─────────── Minute
```

### System cron (one extra field):

```
* * * * * user command
│ │ │ │ │  │
│ │ │ │ │  └─── Command (run as this user)
│ │ │ │ └─────── Day of week
│ │ │ └───────── Month
│ │ └─────────── Day of month
│ └───────────── Hour
└─────────────── Minute
```

> 💡 **System cron** adds a **user field** between the day-of-week and the command.

---

## 2. System Cron Locations

| Location | Purpose | Editable By |
|----------|---------|-------------|
| `/etc/crontab` | Main system crontab | Root (but avoid editing) |
| `/etc/cron.d/` | **Drop-in files** — recommended for custom jobs | Root |
| `/etc/cron.hourly/` | Scripts run **every hour** | Root |
| `/etc/cron.daily/` | Scripts run **every day** | Root |
| `/etc/cron.weekly/` | Scripts run **every week** | Root |
| `/etc/cron.monthly/` | Scripts run **every month** | Root |

> 💡 **Best practice:** Use `/etc/cron.d/` for custom system cron jobs.

---

## 3. Example System Cron Job

### Create a file in `/etc/cron.d/`:

```bash
sudo vi /etc/cron.d/document-backup
```

**Content:**
```
# Backup user Documents at midnight every day
0 0 * * * root /usr/local/bin/document-backup.sh
```

### Breakdown:

| Field | Value | Meaning |
|-------|-------|---------|
| Minute | `0` | At minute 0 |
| Hour | `0` | At midnight |
| Day of month | `*` | Every day |
| Month | `*` | Every month |
| Day of week | `*` | Every day |
| **User** | `root` | Run as root |
| Command | `/usr/local/bin/document-backup.sh` | The script |

---

## 4. Cron vs Systemd Timers — Missing Jobs

### The problem:

If the system was **down at midnight**, a cron job scheduled for midnight **will not run**.

| Method | Handles missed jobs? |
|--------|---------------------|
| Cron (system/user) | ❌ No — job is skipped |
| Systemd timer (with `Persistent=true`) | ✅ Yes — runs on next boot |
| Anacron | ✅ Yes — runs when system is next available |

> 💡 **Systemd timers** with `Persistent=true` are more robust than standard cron for missed schedules.

---

## 5. Anacron — The Missing Piece

- Anacron works **with** cron (they complement each other)
- Anacron ensures jobs run even if the system was down
- Cron calls Anacron via `/etc/cron.hourly/0anacron`

### Anacron configuration — `/etc/anacrontab`:

```
# period delay job-identifier command
START_HOURS_RANGE=3-22
RANDOM_DELAY=45

1       5       cron.daily      nice run-parts /etc/cron.daily
7       10      cron.weekly     nice run-parts /etc/cron.weekly
30      15      cron.monthly    nice run-parts /etc/cron.monthly
```

### Fields:

| Field | Meaning | Example |
|-------|---------|---------|
| Period | How often (in days) | `1` (daily), `7` (weekly), `30` (monthly) |
| Delay | Base delay in minutes | `5`, `10`, `15` |
| Job ID | Unique identifier | `cron.daily` |
| Command | What to run | `nice run-parts /etc/cron.daily` |

---

## 6. Anacron Variables

| Variable | Meaning | Example |
|----------|---------|---------|
| `START_HOURS_RANGE` | Hours when jobs can run | `3-22` (3 AM – 10 PM) |
| `RANDOM_DELAY` | Random delay added (minutes) | `45` |

> 💡 `RANDOM_DELAY` prevents all jobs from running at exactly the same time.

---

## 7. `run-parts` — Running Scripts in a Directory

- `run-parts` executes **all executables** in a directory
- Used by Anacron to process `/etc/cron.daily/`, `/etc/cron.weekly/`, etc.

### Example:

```bash
run-parts /etc/cron.daily
```

**Behavior:** Runs every executable script found in that directory.

---

## 8. Scripts in Directories — Simple Approach

### Step 1 — Create a script:

```bash
sudo vi /etc/cron.daily/backup-documents
```

```bash
#!/bin/bash
/usr/local/bin/document-backup.sh
```

### Step 2 — Make it executable:

```bash
sudo chmod +x /etc/cron.daily/backup-documents
```

### Step 3 — It will run **every day** (managed by Anacron).

> 💡 This is the simplest way to schedule daily/weekly/monthly system tasks.

---

## 9. Cron vs Anacron vs Systemd Timers

| Feature | User Cron | System Cron | Anacron | Systemd Timer |
|---------|-----------|-------------|---------|---------------|
| Syntax | 5 fields | 6 fields (with user) | Period + delay | `OnCalendar` |
| Missed jobs | ❌ No | ❌ No | ✅ Yes | ✅ Yes (with `Persistent`) |
| Per-user jobs | ✅ Yes | ❌ No | ❌ No | ✅ Yes |
| System-wide jobs | ❌ No | ✅ Yes | ✅ Yes | ✅ Yes |
| Integration | Separate daemon | Separate daemon | Called by cron | Built into systemd |
| Configuration | `crontab -e` | `/etc/cron.d/` | `/etc/anacrontab` | `.timer` unit files |

---

## 10. Viewing System Cron Files

### Main system crontab:

```bash
cat /etc/crontab
```

### Drop-in system cron jobs:

```bash
ls -l /etc/cron.d/
cat /etc/cron.d/0hourly
```

### Anacron configuration:

```bash
cat /etc/anacrontab
```

### Script directories:

```bash
ls -l /etc/cron.hourly/
ls -l /etc/cron.daily/
ls -l /etc/cron.weekly/
ls -l /etc/cron.monthly/
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View main system crontab | `cat /etc/crontab` |
| View drop-in system cron files | `ls -l /etc/cron.d/` |
| View anacron config | `cat /etc/anacrontab` |
| View hourly scripts | `ls -l /etc/cron.hourly/` |
| View daily scripts | `ls -l /etc/cron.daily/` |
| View weekly scripts | `ls -l /etc/cron.weekly/` |
| View monthly scripts | `ls -l /etc/cron.monthly/` |
| Run all scripts in directory | `run-parts /etc/cron.daily` |
| View anacron man page | `man 5 anacrontab` |

---

# System Cron — Comparison Table

| Feature | `/etc/crontab` | `/etc/cron.d/*` | `/etc/cron.{hourly,daily,...}` |
|---------|---------------|-----------------|-------------------------------|
| Editable | Root | Root | Root |
| User field | Yes | Yes | No (scripts run as root) |
| Recommended | ❌ Avoid editing | ✅ Yes | ✅ Yes (for simple scripts) |
| Syntax complexity | Full cron syntax | Full cron syntax | No syntax (just scripts) |
| Missed jobs | ❌ No | ❌ No | ✅ Yes (via Anacron) |

---

# Anacron Configuration — Field Details

```
period delay job-identifier command
```

### Example:

```
1       5       cron.daily      nice run-parts /etc/cron.daily
```

| Part | Value | Meaning |
|------|-------|---------|
| Period | `1` | Run every 1 day |
| Delay | `5` | Wait 5 minutes before running |
| Job ID | `cron.daily` | Unique identifier (for tracking) |
| Command | `nice run-parts /etc/cron.daily` | Run all scripts in that directory |

---

# Key Takeaways

- **System cron** adds a **user field** to the cron syntax (6 fields total).
- **Best practice:** Use `/etc/cron.d/` for custom system cron jobs, not `/etc/crontab`.
- **Anacron** ensures jobs run even if the system was down at the scheduled time.
- **`/etc/anacrontab`** defines how `cron.hourly`, `cron.daily`, `cron.weekly`, and `cron.monthly` are processed.
- **`START_HOURS_RANGE`** controls when Anacron jobs can run (default: 3 AM – 10 PM).
- **`RANDOM_DELAY`** spreads jobs out to avoid system load spikes.
- **`run-parts`** executes all executables in a directory.
- **Simplest approach:** Drop an executable script into `/etc/cron.daily/` and it runs every day.
- **Systemd timers** (with `Persistent=true`) are more robust than standard cron for handling missed jobs.
- **Cron + Anacron** work together — cron calls Anacron to ensure reliability.