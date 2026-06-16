# Scheduling Recurring User Jobs — Notes

## 1. What Is Cron?

- **Cron** = scheduler for recurring jobs
- Runs jobs **on a schedule** (not one-time like `at`)
- Jobs run **indefinitely** (into perpetuity)
- **`crond`** = the cron daemon

### Check if crond is running:

```bash
systemctl --no-pager status crond
```

---

## 2. Crontab — Cron Table

- **`crontab`** = **cron** table (timetable of commands)
- Each user can have their own crontab

### Basic crontab commands:

| Command | Purpose |
|---------|---------|
| `crontab -l` | **L**ist current cron jobs |
| `crontab -e` | **E**dit crontab |
| `crontab -r` | **R**emove crontab |

---

## 3. Cron Time Specification — 5 Fields

```
* * * * * command_to_run
│ │ │ │ │
│ │ │ │ └─── Day of week (0-7) [0 or 7 = Sunday, 1 = Monday, 6 = Saturday]
│ │ │ └───── Month (1-12)
│ │ └─────── Day of month (1-31)
│ └───────── Hour (0-23)
└─────────── Minute (0-59)
```

> 💡 **0** and **7** both mean Sunday in the day-of-week field.

---

## 4. Special Characters in Cron

| Character | Meaning | Example |
|-----------|---------|---------|
| `*` | Every instance (all values) | `*` in minute = every minute |
| `,` | List of values | `1,15,30` = at 1, 15, and 30 |
| `-` | Range of values | `1-5` = Monday to Friday |
| `/` | Step values | `*/15` = every 15 minutes |
| `@` | Special strings | `@daily`, `@hourly` |

### Examples:

| Pattern | Meaning |
|---------|---------|
| `*/15 * * * *` | Every 15 minutes |
| `0 2,6,20 * * 1-5` | At 2:00, 6:00, and 20:00 on weekdays |
| `0 5 * * 1` | At 5:00 AM every Monday |
| `0 0 1 * *` | At midnight on the 1st of every month |

---

## 5. Special @ Strings

| String | Meaning | Equivalent |
|--------|---------|------------|
| `@reboot` | Run once at boot | (no equivalent) |
| `@hourly` | Every hour | `0 * * * *` |
| `@daily` | Every day at midnight | `0 0 * * *` |
| `@weekly` | Every week on Sunday | `0 0 * * 0` |
| `@monthly` | Every month on the 1st | `0 0 1 * *` |
| `@yearly` | Every year on Jan 1 | `0 0 1 1 *` |

### Example:

```bash
@daily /usr/bin/backup_script.sh
```

---

## 6. Using `man 5 crontab`

### Why section 5?

| Section | Content |
|---------|---------|
| `man crontab` (section 1) | The **command** `crontab` (how to use `-e`, `-l`, `-r`) |
| `man 5 crontab` (section 5) | The **file syntax** (time fields, special characters, examples) |

> 💡 **`man 5 crontab`** is where you find the syntax and examples — essential for remembering the fields.

### Search for examples:

```bash
man 5 crontab
/EXAMPLE
```

**Output includes:**
```
# run five minutes after midnight, every day
5 0 * * * $HOME/bin/daily.job

# run at 2:15pm on the first of every month
15 14 1 * * $HOME/bin/monthly.job

# run at 10:30am on Monday, Wednesday, and Friday
30 10 * * 1,3,5 $HOME/bin/three_times_week.job
```

---

## 7. Editing Crontabs — `crontab -e`

### Opens with your default editor:

```bash
crontab -e
```

### Set default editor:

```bash
export EDITOR=/usr/bin/vim
```

or

```bash
export EDITOR=/usr/bin/nano
```

> 💡 Add this to `~/.bashrc` for persistence.

---

## 8. Viewing Crontabs — `crontab -l`

```bash
crontab -l
```

**Output:**
```
*/15 * * * * /usr/bin/logger "Running every 15 minutes"
0 2,6,20 * * 1-5 /usr/bin/backup.sh
```

---

## 9. Removing Crontabs — `crontab -r`

```bash
crontab -r
```

> ⚠️ This removes **all** cron jobs for the current user. Use with caution.

---

## 10. System Crontabs vs User Crontabs

| Type | Location | Editable By | Syntax |
|------|----------|-------------|--------|
| User crontabs | `/var/spool/cron/` | Each user (via `crontab -e`) | 5 fields + command |
| System crontabs | `/etc/crontab`, `/etc/cron.d/` | Root only | **6 fields** (includes user field) |

> 📌 System crontabs are **beyond the scope** of this course — focus on user crontabs.

---

## 11. Common Cron Examples

| Requirement | Cron Entry |
|-------------|------------|
| Every 15 minutes | `*/15 * * * * command` |
| Every hour at minute 0 | `0 * * * * command` |
| Every day at 3:00 AM | `0 3 * * * command` |
| Every Monday at 5:00 AM | `0 5 * * 1 command` |
| Every weekday at 9:00 AM | `0 9 * * 1-5 command` |
| First of month at midnight | `0 0 1 * * command` |
| Every 30 minutes, 8 AM to 6 PM | `*/30 8-18 * * * command` |

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Check crond status | `systemctl --no-pager status crond` |
| List cron jobs | `crontab -l` |
| Edit cron jobs | `crontab -e` |
| Remove cron jobs | `crontab -r` |
| View crontab syntax | `man 5 crontab` |
| View crontab command | `man crontab` |
| Search examples | `man 5 crontab` then `/EXAMPLE` |

---

# Cron Field Reference

| Field | Allowed Values | Special Characters |
|-------|----------------|-------------------|
| Minute | 0-59 | `* , - /` |
| Hour | 0-23 | `* , - /` |
| Day of month | 1-31 | `* , - /` |
| Month | 1-12 | `* , - /` |
| Day of week | 0-7 (0 and 7 = Sunday) | `* , - /` |

---

# Key Takeaways

- **Cron** schedules **recurring** jobs (unlike `at` which is one-time).
- **`crond`** is the daemon that runs cron jobs.
- **`crontab -e`** = edit jobs, **`-l`** = list, **`-r`** = remove.
- **5 fields** in user crontabs: minute, hour, day-of-month, month, day-of-week.
- **`*`** = every instance, **`,`** = list, **`-`** = range, **`/`** = step.
- **`man 5 crontab`** = the syntax reference (use it for exams).
- **Section 5** = configuration file syntax (vs section 1 = command).
- **`@` strings** (`@daily`, `@hourly`) are convenient shortcuts.
- **User crontabs** are per-user (`crontab -e`).
- **System crontabs** are system-wide (root only, beyond scope).
- **Exam tip:** Use `man 5 crontab` and search for `/EXAMPLE` to find sample cron jobs.