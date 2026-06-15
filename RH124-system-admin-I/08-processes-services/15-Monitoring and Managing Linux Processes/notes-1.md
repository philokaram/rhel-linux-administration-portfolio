# Processes and the Process Lifecycle — Notes

## 1. What Is a Process?

- **Process** = a program in execution
- Every command, background service, application window = a process

### How a process is created:

1. Executable loaded from **disk** (storage subsystem)
2. Loaded into **memory** (RAM)
3. Kernel assigns a **Process ID (PID)**

---

## 2. Parent-Child Relationships

### Every process has:

| Attribute | Meaning |
|-----------|---------|
| **PID** | Process ID (unique identifier) |
| **PPID** | Parent Process ID (who started this process) |

### The first process — `systemd`:

- **PID = 1**
- PPID = 0 (no parent)
- All processes are **descendants** of `systemd`

### View process tree:

```bash
pstree -p
```

Pipe to `less` for easier navigation:

```bash
pstree -p | less
```

Search for a specific process (e.g., `bash`):

```bash
pstree -p | grep bash
```

**Example output fragment:**
```
systemd(1)───sshd(1234)───sshd-session(1235)───bash(1236)───sudo(1240)───bash(1241)
```

---

## 3. Process States

| State Code | Name | Meaning |
|------------|------|---------|
| `R` | Running | Actively using CPU cycles |
| `S` | Sleeping | Waiting for input or resources (e.g., `sshd` waiting for connections) |
| `T` | Stopped | Paused by system or user |
| `Z` | Zombie | Process finished but not yet cleaned up by parent |

### View a process state:

```bash
sleep 1m &           # Run sleep in background
ps -o pid,comm,stat -p $!   # Show PID, command, state
```

| Part | Meaning |
|------|---------|
| `$!` | PID of the most recently backgrounded process |
| `-o` | Custom output format |
| `pid` | Process ID |
| `comm` | Command name |
| `stat` | Process state |

**Output:** `S` (sleeping)

---

## 4. Viewing Processes — The `ps` Command

### Basic usage — current session only:

```bash
ps
```

### Full format (`-f`):

```bash
ps -f
```

| Column | Meaning |
|--------|---------|
| `UID` | User ID (owner) |
| `PID` | Process ID |
| `PPID` | Parent Process ID |
| `C` | CPU usage |
| `STIME` | Start time |
| `TTY` | Terminal ( `?` = no terminal → daemon) |
| `TIME` | Cumulative CPU time consumed |
| `CMD` | Command |

### All processes with full format:

```bash
ps -e
```

```bash
ps -ef | less
```

### Classic Unix style:

```bash
ps -ef
```

**Note:** Processes with `?` in TTY column = **daemons** (not attached to a terminal).

### Kernel threads:

In `ps -ef` output, look for **square brackets** `[...]`:

```bash
ps -ef | grep "["
```

> 💡 Kernel threads run in kernel memory space, not started by a binary.

---

## 5. Customizing `ps` Output — `ps -eo`

### Select specific columns:

```bash
ps -eo pid,ppid,ni,user,stat,psr,comm
```

| Column | Meaning |
|--------|---------|
| `pid` | Process ID |
| `ppid` | Parent Process ID |
| `ni` | Nice value (priority — covered in RH134) |
| `user` | Owner username |
| `stat` | Process state |
| `psr` | CPU number (0, 1, 2, ...) |
| `comm` | Command name |

### Pipe to `less` for large output:

```bash
ps -eo pid,ppid,ni,user,stat,psr,comm | less
```

> 💡 More than one character in `stat` = sub-status. See student guide for details.

---

## 6. Live Process Monitoring — The `top` Command

### Start top:

```bash
top
```

### What `top` shows:

| Section | Content |
|---------|---------|
| System summary (top lines) | Load averages, memory utilization, CPU usage |
| Process list | Constantly updating, sorted by CPU (default) |

### Interactive `top` commands:

| Key | Action |
|-----|--------|
| `P` (uppercase) | Sort by CPU usage (default) |
| `M` (uppercase) | Sort by memory usage |
| `h` (lowercase) | Help screen |
| `q` | Quit `top` |

> 💡 **Muscle memory tip:** `M` = Memory, `P` = CPU (Process).

---

## 7. Process Lifecycle Summary

```
Program on Disk
       │
       ▼ (execution)
Loaded into Memory
       │
       ▼ (kernel assigns)
Process Created (PID assigned)
       │
       ├──→ Running (R) ──→ Sleep (S) ──→ Running
       │
       ├──→ Stopped (T)
       │
       └──→ Zombie (Z) → Cleared by parent
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View current session processes | `ps` |
| View full format | `ps -f` |
| View all processes | `ps -e` or `ps -ef` |
| View process tree | `pstree -p` |
| Search in process tree | `pstree -p \| grep bash` |
| Custom output columns | `ps -eo pid,ppid,user,stat,comm` |
| Show specific CPU | `ps -eo pid,psr,comm` |
| Show state of background process | `ps -o pid,comm,stat -p $!` |
| Live process view | `top` |
| Sort by memory in top | `M` (uppercase) |
| Sort by CPU in top | `P` (uppercase) |
| Quit top | `q` |

---

# Process State Codes Reference

| Code | Name | Description |
|------|------|-------------|
| `R` | Running | Actively using CPU |
| `S` | Sleeping | Waiting (interruptible) |
| `D` | Sleeping | Waiting (uninterruptible — usually I/O) |
| `T` | Stopped | Paused (by signal) |
| `Z` | Zombie | Terminated but not reaped by parent |

### Sub-status indicators (additional characters):

| Character | Meaning |
|-----------|---------|
| `<` | High priority |
| `N` | Low priority |
| `L` | Has pages locked in memory |
| `s` | Session leader |

---

# `ps` Output Customization — Common Columns

| Column | Description |
|--------|-------------|
| `pid` | Process ID |
| `ppid` | Parent Process ID |
| `user` | Owner username |
| `uid` | Owner UID |
| `group` | Group name |
| `gid` | Group ID |
| `stat` | Process state |
| `psr` | CPU number (0,1,2...) |
| `ni` | Nice value (priority) |
| `time` | Cumulative CPU time |
| `etime` | Elapsed time since start |
| `comm` | Command name (no arguments) |
| `args` | Full command line with arguments |
| `tty` | Terminal ( `?` = daemon) |

---

# Key Takeaways

- **Process** = program in execution. Kernel assigns a **PID**.
- **`systemd`** = PID 1 — ancestor of all processes.
- **`pstree -p`** shows parent-child relationships.
- **Process states:** `R` (running), `S` (sleeping), `T` (stopped), `Z` (zombie).
- **`ps`** = snapshot of processes.
  - `ps -f` = full format (PID, PPID, etc.)
  - `ps -ef` = all processes, full format
  - `ps -eo` = custom columns
- **`top`** = live, updating view of processes.
  - In `top`: `M` = sort by memory, `P` = sort by CPU, `q` = quit
- **`?` in TTY column** = daemon (no terminal attached).
- **Square brackets `[...]`** in `CMD` column = kernel thread.
- **`$!`** = PID of the most recently backgrounded process.
- Work smarter, not harder — **use man pages** (`man ps`, `man top`).