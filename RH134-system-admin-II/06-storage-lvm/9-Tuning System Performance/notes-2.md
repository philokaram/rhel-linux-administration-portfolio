# Influencing Process Scheduling — Notes

## 1. The Scheduler's Job

The Linux kernel uses a **scheduler** to manage which process gets to use the CPU.

### Scheduling Policies:

| Policy | Purpose |
|--------|---------|
| **SCHED_FIFO** (Real-time) | Highest priority process runs to completion without interruption |
| **SCHED_RR** (Real-time) | Fixed time slice, then moves to back of queue |
| **SCHED_OTHER** (Normal) | Default for everyday applications — aims for fairness |

### RHEL 10 — EEVDF (Earliest Eligible Virtual Deadline First)

- Improves upon the Completely Fair Scheduler (CFS)
- Adds **deadlines** to tasks → prioritizes not just waiting time, but **urgency**
- Results in a more responsive system

---

## 2. Nice Values — Influencing Priority

- A way to tell the kernel **how nice** a process should be to others
- Range: **-20** (least nice, highest priority) to **+19** (nicest, lowest priority)

### Nice Value Rules:

| Value | Meaning | Who can set |
|-------|---------|-------------|
| **-20** | **Less nice** → higher priority (more CPU) | Only **root** |
| 0 | Default priority | Any user |
| **+19** | **Very nice** → lower priority (less CPU) | Any user (can increase) |

> 💡 **Unprivileged users can only increase** their nice value (make processes nicer). Only root can decrease nice values.

---

## 3. Viewing Nice Values — `top`

### In `top`, look for the `NI` column:

```bash
top
```

| Column | Meaning |
|--------|---------|
| `NI` | Nice value of the process |
| `%CPU` | CPU usage percentage |
| `PID` | Process ID |

> 💡 Default nice value is **0**.

---

## 4. Starting a Process with a Nice Value — `nice`

```bash
nice -n -10 dd if=/dev/zero of=/dev/null &
```

| Option | Meaning |
|--------|---------|
| `-n -10` | Set nice value to **-10** (higher priority) |
| `dd if=/dev/zero of=/dev/null` | CPU-intensive command (zeroes to black hole) |

### Without nice value (default 0):

```bash
dd if=/dev/zero of=/dev/null &
```

> ⚠️ **Warning:** `dd if=/dev/zero of=/dev/null` consumes 100% CPU. **Never run in production.**

---

## 5. Changing Nice Value of Running Process — `renice`

```bash
renice -n -20 62816
```

| Option | Meaning |
|--------|---------|
| `-n -20` | Set nice value to **-20** (highest priority) |
| `62816` | Process ID of the target process |

### Increase nice value (lower priority) — any user can do this:

```bash
renice -n +19 62727
```

---

## 6. CPU Contention Example — Observing Nice Values in Action

### Scenario:

- 4 `dd` processes running (all want 100% CPU)
- 2 CPUs available

### Results:

| Nice Value | CPU Time | Behavior |
|------------|----------|----------|
| **-20** | ~90-100% | Highest priority, gets most CPU |
| **-10** | ~80-90% | High priority, shares with other low-priority processes |
| **0** | ~10% | Default, gets less when higher-priority processes compete |
| **+19** | ~0.3% | Lowest priority, barely gets any CPU time |

> 💡 Processes with higher priority (lower nice value) get significantly more CPU time under contention.

---

## 7. Setting Nice Values for System Services — systemd Override

### Problem:

- You want a service (like `httpd`) to always start with a specific nice value

### Solution — Systemd Drop-in:

**Step 1 — Create drop-in directory:**

```bash
mkdir -p /etc/systemd/system/httpd.service.d
```

**Step 2 — Create override file:**

```bash
vi /etc/systemd/system/httpd.service.d/99-nice.conf
```

**Content:**
```ini
[Service]
ExecStart=
ExecStart=/usr/bin/nice -n -10 /usr/sbin/httpd -DFOREGROUND
```

| Part | Meaning |
|------|---------|
| `ExecStart=` | **Clear** the original ExecStart |
| `ExecStart=...` | **Replace** with new command using `nice` |

**Step 3 — Reload systemd:**

```bash
systemctl daemon-reload
```

**Step 4 — Restart the service:**

```bash
systemctl restart httpd
```

### Verify:

```bash
ps -axo pid,ni,comm | grep httpd
```

**Expected:** `NI` column shows `-10` for all `httpd` processes.

> 💡 Never edit `/usr/lib/systemd/system/httpd.service` directly — use drop-in files.

---

## 8. Viewing Nice Values and CPU Affinity

```bash
ps -axo pid,ni,psr,comm | grep httpd
```

| Column | Meaning |
|--------|---------|
| `pid` | Process ID |
| `ni` | Nice value |
| `psr` | CPU number the process is running on |
| `comm` | Command name |

---

## 9. Killing All `dd` Processes

```bash
killall dd
```

> 💡 Clean up CPU-hungry processes after testing.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View nice values (top) | `top` (look for `NI` column) |
| View nice values (ps) | `ps -axo pid,ni,comm` |
| Start process with nice value | `nice -n -10 dd if=/dev/zero of=/dev/null &` |
| Change nice value of running process | `renice -n -20 62816` |
| Increase nice value (lower priority) | `renice -n +19 62727` |
| View number of CPUs | `nproc` |
| Kill all dd processes | `killall dd` |
| Create systemd drop-in | `mkdir -p /etc/systemd/system/service.d/` |
| Reload systemd config | `systemctl daemon-reload` |
| Restart service | `systemctl restart httpd` |

---

# Nice Values — Quick Reference

| Nice Value | Priority | CPU Time (under contention) | Who can set |
|------------|----------|-----------------------------|-------------|
| -20 | Highest | ~90-100% | Root only |
| -10 | High | ~80-90% | Root only |
| 0 | Default | ~10% | Any user |
| +10 | Low | ~1% | Any user |
| +19 | Lowest | ~0.3% | Any user |

---

# Systemd Drop-in Override — Pattern

```
/usr/lib/systemd/system/service.service   ← System file (do not edit)
                    │
                    ▼
/etc/systemd/system/service.service.d/     ← Custom drop-in directory
└── 99-custom.conf                          ← Override file
```

### Override file syntax:

```ini
[Service]
ExecStart=               # Clear original
ExecStart=/usr/bin/nice -n -10 /usr/sbin/httpd -DFOREGROUND  # New command
```

---

# Key Takeaways

- **Linux scheduler** manages CPU time allocation between processes.
- **Nice values** (-20 to +19) influence priority:
  - Lower = **less nice** = higher priority
  - Higher = **more nice** = lower priority
- **Unprivileged users** can only increase nice values (make processes nicer).
- **Root** can decrease nice values (increase priority).
- **`nice`** starts a process with a specific nice value.
- **`renice`** changes the nice value of a running process.
- **Systemd drop-in files** can override service settings without editing the original unit file.
- **CPU contention** reveals the impact of nice values:
  - High priority processes get more CPU time
  - Low priority processes barely get any
- **Exam tip:** Remember the nice value range (-20 to +19) and who can set what.
- **Warning:** `dd if=/dev/zero of=/dev/null` is CPU-intensive — only use for testing, never in production.