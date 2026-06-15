# Monitoring Process Activity — Notes

## 1. What Is Load Average?

- The **average number of processes** that are:
  - **Running** on the CPU, or
  - **Waiting** to run on the CPU

> 📌 **Load average is NOT CPU percentage.** It's about how many tasks are competing for CPU time.

### Where to see load average:

| Command | Load Average Location |
|---------|----------------------|
| `uptime` | End of output (snapshot) |
| `top` | Top line (live updating) |

### The three numbers:

| Position | Time Period | Use Case |
|----------|-------------|----------|
| 1st number | Last **1 minute** | Detects short-term spikes |
| 2nd number | Last **5 minutes** | Shows recent trend |
| 3rd number | Last **15 minutes** | Indicates sustained load |

> 💡 Compare the three numbers to see if load is **spiking briefly** or **sustained over time**.

---

## 2. Interpreting Load Average — You MUST Know Your CPU Count

### Find number of CPUs:

```bash
nproc
```

### Interpretation rules:

| Load Value (on 1 CPU system) | Meaning |
|------------------------------|---------|
| `1.00` | Fully busy (100% utilization) |
| `< 1.00` | Some spare capacity |
| `> 1.00` | Some tasks waiting (overloaded) |

### On a system with **2 CPUs**:

| Load Value | Meaning |
|------------|---------|
| `2.00` | Both CPUs fully busy (perfect balance) |
| `< 2.00` | Spare capacity |
| `> 2.00` | Overloaded — some tasks waiting |

> 💡 **Key formula:** Compare load average to number of CPUs.
> - Load < CPU count = spare capacity
> - Load = CPU count = fully utilized
> - Load > CPU count = overloaded (tasks waiting)

---

## 3. Understanding Load Average Decimals

- You never have `0.29` of a process
- The decimal represents the **fraction of time** a process was queued

### Example:

Load average of `2.29` on a 2-CPU system means:
- CPUs were **always busy** (2.00)
- For about **29% of the time**, there was **1 extra process** waiting

### Don't panic — context matters:

| Scenario | CPU Count | Load = 40 | Interpretation |
|----------|-----------|-----------|----------------|
| Small server | 2 CPUs | 40 | **Severe overload** |
| Large server | 80 CPUs | 40 | **Underutilized** (spare capacity) |

> 💡 Always know your CPU count before interpreting load average.

---

## 4. The `uptime` Command — Load Average Snapshot

```bash
uptime
```

**Output example:**
```
14:30:01 up 2 days, 3:45, 3 users, load average: 2.29, 2.02, 1.20
```

| Field | Meaning |
|-------|---------|
| `14:30:01` | Current time |
| `up 2 days, 3:45` | System uptime |
| `3 users` | Number of logged-in users |
| `load average: 2.29, 2.02, 1.20` | Load for 1, 5, 15 minutes |

---

## 5. The `top` Command — Live Process View

```bash
top
```

### What `top` shows (summary area):

| Line | Content |
|------|---------|
| 1 | Time, uptime, users, load average |
| 2 | Tasks (total, running, sleeping, stopped, zombie) |
| 3 | CPU usage (us, sy, ni, id, wa, hi, si, st) |
| 4 | Memory usage |
| 5 | Swap usage |

### Process list area:

- Sorted by **CPU usage by default** (highest first)
- Updates every **2 seconds** (default interval)

### Important process information in `top`:

| Column | Meaning |
|--------|---------|
| `PID` | Process ID |
| `USER` | Owner |
| `PR` | Priority |
| `NI` | Nice value |
| `VIRT` | Virtual memory used |
| `RES` | Resident memory (physical RAM) |
| `SHR` | Shared memory |
| **`S`** | **Process state** (R=Running, S=Sleeping, D=Uninterruptible, Z=Zombie) |
| `%CPU` | CPU usage percentage |
| `%MEM` | Memory usage percentage |
| `TIME+` | Cumulative CPU time |
| `COMMAND` | Command name |

---

## 6. Killing a Process from Within `top`

### Steps:

1. Press `k` (kill)
2. Enter PID (default = top process if you press Enter)
3. Enter signal number (default = `15` = SIGTERM)
4. Press Enter

### Example:

```
top
# Press k
PID to kill: 235770
# Press Enter (accepts default)
Send signal [15]: 
# Press Enter (accepts default)
```

> 💡 Default signal in `top` is **SIGTERM (15)** — graceful termination.

---

## 7. Quitting `top`

| Key | Action |
|-----|--------|
| `q` | Quit top |
| `Ctrl+C` | Also works, but `q` is proper |

> ⚠️ Don't use `Ctrl+C` out of habit — `q` is the correct way to quit `top`.

---

## 8. CPU-Hungry Process Demonstration (WARNING)

### Dangerous command — **NEVER run in production**:

```bash
dd if=/dev/zero of=/dev/null
```

| Part | Meaning |
|------|---------|
| `dd` | Disk/data duplicator |
| `if=/dev/zero` | Input file = infinite zeros |
| `of=/dev/null` | Output file = black hole (discard data) |

**Effect:** Consumes 100% of one CPU core continuously.

### On a 2-CPU system, run 3 instances:

```bash
dd if=/dev/zero of=/dev/null &   # Run in background
dd if=/dev/zero of=/dev/null &
dd if=/dev/zero of=/dev/null &
```

### Observe the load average climb:

```bash
uptime
```

The load will exceed the CPU count, indicating overload.

---

## 9. Process States in `top`

| State | Meaning |
|-------|---------|
| `R` | Running (currently on CPU) |
| `S` | Sleeping (waiting for event) |
| `D` | Uninterruptible sleep (usually I/O) |
| `Z` | Zombie (terminated, not reaped) |
| `T` | Stopped (suspended) |

> 💡 Only processes in **`R` state** consume CPU cycles.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View load average (snapshot) | `uptime` |
| View number of CPUs | `nproc` |
| Live process monitoring | `top` |
| Kill process from top | Press `k`, enter PID, enter signal |
| Quit top | Press `q` |
| Run CPU-intensive test (dangerous) | `dd if=/dev/zero of=/dev/null &` |

---

# Load Average Interpretation — Quick Guide

| Comparison | Meaning |
|------------|---------|
| Load < CPU count | Spare capacity — system underutilized |
| Load = CPU count | Fully utilized — perfect balance |
| Load > CPU count | Overloaded — tasks waiting for CPU |

### Example scenarios:

| CPUs | Load (1 min) | Interpretation |
|------|--------------|----------------|
| 1 | 0.50 | 50% idle |
| 1 | 1.00 | Fully busy |
| 1 | 1.50 | Overloaded (50% of time, 1 task waiting) |
| 2 | 1.80 | Spare capacity (20% idle) |
| 2 | 2.00 | Both CPUs fully busy |
| 2 | 3.50 | Overloaded (1.5 tasks waiting on average) |
| 80 | 40.00 | Underutilized (50% idle) |

---

# `top` Summary Area — Line-by-Line

```
top - 14:30:01 up 2 days, 3:45, 3 users, load average: 2.29, 2.02, 1.20
Tasks: 120 total, 3 running, 117 sleeping, 0 stopped, 0 zombie
%Cpu(s): 85.2 us, 10.5 sy, 0.0 ni, 4.0 id, 0.0 wa, 0.0 hi, 0.3 si, 0.0 st
MiB Mem : 7812.5 total, 234.8 free, 5123.1 used, 2454.6 buff/cache
MiB Swap: 2048.0 total, 2048.0 free, 0.0 used. 1456.2 avail Mem
```

| Field | Meaning |
|-------|---------|
| `us` | User space CPU |
| `sy` | System/kernel CPU |
| `id` | Idle CPU |
| `wa` | I/O wait |
| `hi` | Hardware interrupts |
| `si` | Software interrupts |

---

# Key Takeaways

- **Load average** = average number of processes running OR waiting for CPU (not CPU percentage).
- Load average shows **1, 5, and 15 minute** averages — compare them to spot trends.
- **You must know your CPU count** (`nproc`) to interpret load average correctly.
- **Load < CPU count** = spare capacity. **Load > CPU count** = overloaded (tasks waiting).
- Decimals represent the **fraction of time** a process was queued.
- **`uptime`** = snapshot of load average.
- **`top`** = live, updating view of processes and load average.
- In `top`, press **`k`** to kill a process, **`q`** to quit.
- Processes in **`R` state** are actively using CPU.
- The `dd if=/dev/zero of=/dev/null` command is **dangerous** — consumes all CPU cycles.
- **Don't panic at high load average** — if you have 80 CPUs, load of 40 is fine. Context matters.