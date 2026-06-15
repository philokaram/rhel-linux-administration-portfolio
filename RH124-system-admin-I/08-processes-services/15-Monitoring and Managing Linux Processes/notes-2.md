# Managing Processes with Job Control — Notes

## 1. Foreground vs Background

### Foreground (default):

- Command **takes over** your terminal
- You cannot run other commands until it finishes

### Example — foreground command blocks the terminal:

```bash
sleep 10
date          # Won't run until sleep finishes
ls
uname -r
```

### Background (using `&`):

- Command runs in the **background**
- You **get your prompt back** immediately
- Shell returns: `[job_number] process_id`

```bash
sleep 1m &
```

**Output:** `[1] 89552` (job number 1, PID 89552)

### Now you can run other commands:

```bash
date
ls
uname -r
```

---

## 2. Viewing Jobs — `jobs` Command

```bash
jobs
```

**Shows:** Job number, state, and command for all jobs in current shell session.

### Viewing process states — `ps j`:

```bash
ps j
```

| Column | Meaning |
|--------|---------|
| `STAT` | Process state (R=Running, S=Sleeping, T=Stopped) |
| `PID` | Process ID |
| `PGID` | Process Group ID |
| `SESS` | Session ID |
| `COMMAND` | Command name |

> 💡 In `ps j` output, most processes show `S` (sleeping) — only actively running commands show `R`.

---

## 3. Suspending a Foreground Process — `Ctrl+Z`

### Scenario:

1. Start a long-running foreground process (e.g., `gnome-calculator`)
2. You want your prompt back without terminating the process

### Solution — Suspend with `Ctrl+Z`:

```bash
gnome-calculator
# ... calculator is open ...
# Press Ctrl+Z
```

**Output:** `[2]+  Stopped    gnome-calculator`

### What happens:

- Process is **stopped** (state = `T` in `ps j`)
- You get your prompt back
- Process is not running, but still exists

---

## 4. `Ctrl+Z` vs `Ctrl+C` — Critical Difference

| Key Combination | Effect | Process State |
|-----------------|--------|---------------|
| `Ctrl+Z` | **Suspend** (pause) | Stopped (can be resumed) |
| `Ctrl+C` | **Terminate** (kill) | Dead (cannot be resumed) |

> 💡 `Ctrl+Z` = pause. `Ctrl+C` = stop permanently.

---

## 5. Foreground, Background, Resume — `fg` and `bg`

### `fg` — Bring job to foreground:

```bash
fg %job_number
```

Example:

```bash
fg %2        # Bring job #2 to foreground
```

### `bg` — Resume job in background:

```bash
bg %job_number
```

Example:

```bash
bg %1        # Resume job #1 in background
```

### Resume a stopped job as if it was started with `&`:

```bash
gnome-calculator      # Start in foreground
Ctrl+Z                # Suspend
bg %1                 # Resume in background (calculator still works!)
```

---

## 6. Job Numbers vs Process IDs — Important Distinction

| Feature | Job Number | Process ID (PID) |
|---------|------------|------------------|
| Managed by | Shell | Kernel |
| Scope | **Session-specific** (temporary) | **System-wide** |
| Lifespan | Only for current shell session | Until process terminates |
| Use with | `fg`, `bg`, `jobs` | `kill`, `ps`, `top` |

> 💡 Job numbers are **temporary** — different terminals have different job numbers for the same process.

---

## 7. Complete Job Control Workflow Example

```bash
# Start calculator in foreground
gnome-calculator

# Use calculator... then suspend it
Ctrl+Z

# Verify it's stopped
jobs
ps j | grep calculator

# Resume in background
bg %1

# Verify it's now running in background
jobs

# Calculator is still usable! Do math while terminal is free
# When done, bring to foreground and terminate
fg %1
Ctrl+C
```

---

## 8. Process States in Job Control

| State | Symbol | How to get there | How to leave |
|-------|--------|------------------|--------------|
| Running (foreground) | (no symbol) | Normal execution | `Ctrl+Z` (suspend) or `Ctrl+C` (terminate) |
| Running (background) | `&` or `bg` | Start with `&` or resume with `bg` | `fg` to bring to foreground |
| Stopped | `T` (in `ps`) | `Ctrl+Z` | `fg` or `bg` |

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Run command in background | `command &` |
| List jobs in current session | `jobs` |
| View processes with state | `ps j` |
| Suspend foreground process | `Ctrl+Z` |
| Terminate foreground process | `Ctrl+C` |
| Bring job to foreground | `fg %job_number` |
| Resume job in background | `bg %job_number` |

---

# Job Control — Visual Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│                         Shell Session                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Start command normally → FOREGROUND (terminal blocked)         │
│                              │                                   │
│                              ├─── Ctrl+Z → STOPPED (T state)    │
│                              │              │                    │
│                              │              ├── fg → FOREGROUND  │
│                              │              └── bg → BACKGROUND  │
│                              │                                   │
│                              └─── Ctrl+C → TERMINATED (dead)     │
│                                                                  │
│  Start with & → BACKGROUND (prompt returns)                     │
│                              │                                   │
│                              └─── fg → FOREGROUND                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

# Key Takeaways

- **Foreground** commands take over your terminal — no other commands until they finish.
- **Background** (`&`) runs the command and returns the prompt immediately.
- Shell returns `[job_number] PID` when starting a background job.
- **`Ctrl+Z`** = **suspend** (pause) a foreground process — it stops but can be resumed.
- **`Ctrl+C`** = **terminate** a process — it's gone permanently.
- **`jobs`** shows job numbers for the current shell session only.
- **`ps j`** shows process states (`R`, `S`, `T`, etc.).
- **`fg %job_number`** brings a background or stopped job to the foreground.
- **`bg %job_number`** resumes a stopped job in the background.
- **Job numbers** are shell-specific and temporary; **PIDs** are kernel-managed and system-wide.
- Stopped process (`T` state) is not consuming CPU — it's just paused, waiting to be resumed or terminated.