# Scheduling a Future Job — Notes

## 1. What Is the `at` Daemon?

- **`atd`** = the at daemon (runs scheduled one-time jobs)
- **`at` command** = interface to schedule jobs
- Jobs run **once** at a specified time — **never execute again**

### How it works:

```
User runs: at $TIME
         │
         ▼
User enters commands
         │
         ▼
User presses Ctrl+D
         │
         ▼
Job stored in /var/spool/at/
         │
         ▼
atd executes job at specified time (ONE TIME ONLY)
```

---

## 2. Checking if `atd` Is Running

```bash
systemctl status atd
```

### Without pager (returns immediately):

```bash
systemctl --no-pager status atd
```

> 💡 `--no-pager` shows output using `cat` instead of `less` — no need to press `q`.

---

## 3. Scheduling a Job — Basic Example

### Step 1 — Check current time:

```bash
date
```

**Output:** `Mon Sep  1 20:31:39 UTC 2025`

### Step 2 — Schedule a job:

```bash
at 20:33
```

**Output:**
```
warning: commands will be executed using /bin/sh
at>
```

### Step 3 — Enter commands:

```
at> mkdir -p /foo/bar/baz/bongle
at> touch /foo/bar/baz/bongle/result
at> date > /foo/bar/baz/bongle/result
```

### Step 4 — Submit the job:

Press **`Ctrl+D`**

**Output:**
```
job 4 at Mon Sep  1 20:33:00 2025
```

> 💡 **`Ctrl+D`** = exit/logout — submits the job.

---

## 4. Viewing Scheduled Jobs — `atq`

```bash
atq
```

**Output:**
```
4       Mon Sep  1 20:33:00 2025 a student
```

| Column | Meaning |
|--------|---------|
| Job number | `4` |
| Date/time | When the job will run |
| Queue | `a` (default queue) |
| User | `student` |

---

## 5. Removing a Scheduled Job — `atrm`

### Remove a job:

```bash
atrm 4
```

### Verify removal:

```bash
atq
```

**Output:** (nothing — job is gone)

---

## 6. Job Storage Location

Scheduled jobs are stored as files:

```bash
ls -l /var/spool/at/
```

> 📌 Each user has their own jobs stored here.

---

## 7. Time Formats for `at`

| Format | Example | Meaning |
|--------|---------|---------|
| HH:MM | `20:33` | At 8:33 PM today |
| HH:MM AM/PM | `08:33 PM` | At 8:33 PM today |
| midnight | `midnight` | 12:00 AM |
| noon | `noon` | 12:00 PM |
| teatime | `teatime` | 4:00 PM |
| now + N units | `now + 30 minutes` | 30 minutes from now |
| YYYY-MM-DD | `2025-09-01 20:33` | Specific date and time |

### Examples:

```bash
at 15:30
at midnight
at now + 5 minutes
at noon September 2
at 8:30 AM Friday
```

---

## 8. Scheduling Multiple Commands

### Method 1 — Enter commands line by line:

```bash
at 20:33
at> mkdir /tmp/test
at> touch /tmp/test/file
at> echo "Done" > /tmp/test/result
Ctrl+D
```

### Method 2 — Use a here document:

```bash
at 20:33 << EOF
mkdir /tmp/test
touch /tmp/test/file
echo "Done" > /tmp/test/result
EOF
```

### Method 3 — Run a script:

```bash
at 20:33 -f /path/to/script.sh
```

---

## 9. Important Warning — Don't Do This!

```bash
at midnight << EOF
for i in {1..1000}; do
    dd if=/dev/zero of=/dev/null &
done
EOF
```

> ⚠️ **Warning:** This will consume all CPU cycles and bring the system to a grinding halt. **Never run this in production.**

---

## 10. `at` vs `cron`

| Feature | `at` | `cron` |
|---------|------|--------|
| Execution | **One time only** | Recurring (daily, weekly, etc.) |
| Use case | One-off tasks | Regular, scheduled tasks |
| Command | `at` | `crontab` |
| Job storage | `/var/spool/at/` | `/var/spool/cron/` |

> 💡 Use `at` for "run this once at this time." Use `cron` for "run this every day at this time."

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Check if atd is running | `systemctl status atd` |
| Check without pager | `systemctl --no-pager status atd` |
| Schedule a job | `at $TIME` (then enter commands, Ctrl+D) |
| Schedule with script | `at $TIME -f /path/to/script` |
| View scheduled jobs | `atq` |
| Remove a job | `atrm JOB_NUMBER` |
| View job files | `ls -l /var/spool/at/` |

---

# Time Format Examples

| Input | Meaning |
|-------|---------|
| `at 15:30` | 3:30 PM today |
| `at 3:30 PM` | 3:30 PM today |
| `at midnight` | 12:00 AM tonight |
| `at noon` | 12:00 PM today |
| `at now + 30 minutes` | 30 minutes from now |
| `at now + 1 hour` | 1 hour from now |
| `at 2025-12-31 23:59` | Specific date and time |
| `at 8:30 AM Friday` | 8:30 AM on next Friday |

---

# Workflow Example

```
1. Check current time
   $ date
   Mon Sep  1 20:31:39 UTC 2025

2. Schedule job for 20:33
   $ at 20:33
   warning: commands will be executed using /bin/sh
   at> mkdir -p /foo/bar/baz/bongle
   at> touch /foo/bar/baz/bongle/result
   at> date > /foo/bar/baz/bongle/result
   at> <Ctrl+D>
   job 4 at Mon Sep  1 20:33:00 2025

3. Verify job is scheduled
   $ atq
   4       Mon Sep  1 20:33:00 2025 a student

4. Wait for job to execute

5. Verify results
   $ date
   Mon Sep  1 20:33:15 UTC 2025
   $ ls -l /foo/bar/baz/bongle/
   -rw-r--r-- 1 student student 29 Sep  1 20:33 result
   $ cat /foo/bar/baz/bongle/result
   Mon Sep  1 20:33:00 UTC 2025
```

---

# Key Takeaways

- **`at`** schedules **one-time** jobs (never repeat).
- **`atd`** is the daemon that runs scheduled jobs (always running by default).
- **`atq`** lists pending jobs.
- **`atrm`** removes a pending job.
- **`Ctrl+D`** submits the job (exit/logout).
- Jobs are stored in `/var/spool/at/`.
- Time formats: `HH:MM`, `now + N minutes`, `midnight`, `noon`, specific dates.
- `at` is for **one-time** tasks; use **`cron`** for recurring tasks.
- Be careful with commands that consume CPU (e.g., `dd`) — never schedule them in production.
- **Work smarter**: Use `--no-pager` to avoid `less` when you don't need it.