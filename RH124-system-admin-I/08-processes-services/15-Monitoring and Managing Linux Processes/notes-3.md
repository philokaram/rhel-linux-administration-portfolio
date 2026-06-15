# Sending Signals to Processes — Notes

## 1. What Are Signals?

- **Signals** (also called kill signals) = how we manage processes
- Each signal has a **number** and a **name**
- The `kill` command doesn't necessarily mean "kill" — it sends signals

### View all signals:

```bash
kill -l
```

or

```bash
man 7 signal
```

---

## 2. Common Signals — Summary

| Signal Number | Signal Name | Meaning | Default Effect |
|---------------|-------------|---------|----------------|
| 1 | `SIGHUP` | Hang up | **Reload** configuration (process continues with same PID) |
| 9 | `SIGKILL` | Kill | **Force terminate** — cannot be caught/ignored |
| 15 | `SIGTERM` | Terminate | **Graceful termination** (default for `kill`) |
| 18 | `SIGCONT` | Continue | Resume stopped process |
| 19 | `SIGSTOP` | Stop | Pause process (same as `Ctrl+Z`) — cannot be caught/ignored |

> 💡 `Ctrl+C` sends `SIGTERM` (signal 15) — graceful termination.
> 💡 `Ctrl+Z` sends `SIGSTOP` (signal 19) — suspend/pause.

---

## 3. Signal 1 — `SIGHUP` (Reload Configuration)

### Use case:

1. You change a service's configuration file on disk
2. The service already loaded the old config into **memory**
3. Send `SIGHUP` → service reloads config from disk **without restarting**

### Effect:

- Process continues running uninterrupted
- Same Process ID (PID)
- New configuration takes effect

### Command:

```bash
kill -1 <PID>
```

or

```bash
kill -HUP <PID>
```

---

## 4. Signal 15 — `SIGTERM` (Graceful Termination)

- **Default signal** for `kill` (if no signal specified)
- Allows process to clean up before exiting
- Same as `Ctrl+C`

### Command:

```bash
kill <PID>           # Same as kill -15 <PID>
```

or

```bash
kill -15 <PID>
```

or

```bash
kill -TERM <PID>
```

---

## 5. Signal 9 — `SIGKILL` (Force Kill)

- **"Die right now"** — immediate termination
- Process cannot catch or ignore this signal
- **No cleanup** — data loss possible
- Last resort only

### Command:

```bash
kill -9 <PID>
```

or

```bash
kill -KILL <PID>
```

> ⚠️ **Use judiciously** — don't use `-9` too liberally. Try `SIGTERM` (signal 15) first.

---

## 6. Signal 19 — `SIGSTOP` (Pause/Suspend)

- Same as `Ctrl+Z`
- Process is prevented from getting CPU cycles
- Process still exists but is **stopped**

### Command:

```bash
kill -19 <PID>
```

or

```bash
kill -STOP <PID>
```

---

## 7. Signal 18 — `SIGCONT` (Resume)

- Resume a stopped process
- Process gets CPU cycles again
- Same as `fg` or `bg` after `Ctrl+Z`

### Command:

```bash
kill -18 <PID>
```

or

```bash
kill -CONT <PID>
```

---

## 8. Finding Process IDs — `pgrep`

### Basic search:

```bash
pgrep gnome-calculator
```

**Returns:** PID(s) matching the name

### Search with long output (command line):

```bash
pgrep -la gnome-calculator
```

| Option | Meaning |
|--------|---------|
| `-l` | Show process name |
| `-a` | Show full command line |
| `-u` | Filter by user |
| `-t` | Filter by terminal |

### Search by user:

```bash
pgrep -la -u mo
```

### Search by terminal (tty):

```bash
pgrep -la -t pts/1
```

### Search for exact match (no partial matches):

```bash
pgrep -x "dd"
```

---

## 9. Sending Signals by Name — `pkill`

### Kill all processes for a user (DANGEROUS):

```bash
pkill -SIGKILL -u mo
```

> ⚠️ This logs the user out of **all** sessions.

### Kill processes on a specific terminal:

```bash
pkill -SIGKILL -t pts/1
```

### Kill a specific process by name:

```bash
pkill -15 dd        # Graceful termination
pkill -9 dd         # Force kill
```

---

## 10. Killing All Processes by Name — `killall`

### Terminate all processes named `dd`:

```bash
killall dd
```

### With specific signal:

```bash
killall -9 dd
```

> 💡 Default signal for `killall` is also `SIGTERM` (15).

---

## 11. Viewing Process Trees by User — `pstree`

### Show process tree for a specific user:

```bash
pstree -p mo
```

| Option | Meaning |
|--------|---------|
| `-p` | Show PIDs |

**Example output:**
```
mo(233400)───sshd-session(233401)───bash(233402)───dd(233409)
```

---

## 12. Practical Troubleshooting Example

### Problem: System is slow, suspect user `mo` is running a CPU-consuming `dd` command

### Step 1 — Find the offensive process:

```bash
pgrep -la dd
```

**Output:** `233409 dd if=/dev/zero of=/dev/null`

### Step 2 — Try graceful termination first:

```bash
kill 233409
```

### Step 3 — Check if still running:

```bash
pgrep -la dd
```

If still running → process ignored `SIGTERM`.

### Step 4 — Force kill:

```bash
kill -9 233409
```

### Alternative — Log the user out of the problematic session:

```bash
# Find which terminal the naughty process is on
pgrep -la -u mo

# Kill that terminal's session
pkill -9 -t pts/1
```

Other sessions for user `mo` remain unaffected.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| List all signals | `kill -l` |
| Send default signal (SIGTERM) | `kill <PID>` |
| Send SIGTERM explicitly | `kill -15 <PID>` or `kill -TERM <PID>` |
| Send SIGHUP (reload config) | `kill -1 <PID>` or `kill -HUP <PID>` |
| Send SIGKILL (force kill) | `kill -9 <PID>` or `kill -KILL <PID>` |
| Send SIGSTOP (pause) | `kill -19 <PID>` or `kill -STOP <PID>` |
| Send SIGCONT (resume) | `kill -18 <PID>` or `kill -CONT <PID>` |
| Find PID by name | `pgrep <name>` |
| Find PID with full command line | `pgrep -la <name>` |
| Find processes by user | `pgrep -la -u <username>` |
| Find processes by terminal | `pgrep -la -t pts/1` |
| Kill by name | `pkill <name>` |
| Kill by name with specific signal | `pkill -9 <name>` |
| Kill all processes on terminal | `pkill -9 -t pts/1` |
| Kill all processes for user | `pkill -9 -u <username>` |
| Kill all processes by name | `killall <name>` |
| Kill all processes by name with signal | `killall -9 <name>` |
| Show process tree for user | `pstree -p <username>` |

---

# Signals Quick Reference

| Number | Name | Use Case |
|--------|------|----------|
| 1 | `SIGHUP` | Reload configuration (no restart) |
| 9 | `SIGKILL` | Force kill (last resort) |
| 15 | `SIGTERM` | Graceful termination (default) |
| 18 | `SIGCONT` | Resume stopped process |
| 19 | `SIGSTOP` | Pause/suspend process |

---

# Signal Flow — Reload Configuration (SIGHUP)

```
Configuration file on disk (updated)
         │
         │ SIGHUP (kill -1)
         ▼
Running Process (old config in memory)
         │
         │ "Hang up" old connection
         │
         ▼
Process reloads config from disk
         │
         ▼
Process continues with same PID, new config
```

---

# Key Takeaways

- **Signals** = how we manage processes (number + name).
- **`kill` command** doesn't mean "kill" — it sends signals.
- **`kill -l`** lists all signals.
- **`SIGTERM` (15)** = default signal — graceful termination (`Ctrl+C`).
- **`SIGKILL` (9)** = force kill — last resort only.
- **`SIGHUP` (1)** = reload configuration without restarting (process keeps same PID).
- **`SIGSTOP` (19)** = pause process (`Ctrl+Z`).
- **`SIGCONT` (18)** = resume paused process.
- **`pgrep`** finds PIDs by name, user, terminal.
- **`pkill`** sends signals by name, user, terminal.
- **`killall`** kills all processes with a given name.
- **`pstree -p`** shows process hierarchy with PIDs.
- Be judicious with `SIGKILL` — try `SIGTERM` first.
- To log a user out of a specific session, use `pkill -9 -t pts/N`.