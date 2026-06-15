# Controlling System Services — Notes

## 1. Review: systemd Basics

- **systemd** = initialization system + service manager (created by Red Hat, shared with all Linux distros)
- **PID 1** = first process after kernel boots
- Starts: background services, login prompts, system targets
- Features: consistent service management, dependency handling, monitoring, logging, troubleshooting

> 💡 Open source win: Red Hat created systemd and shared it with everyone.

---

## 2. Where Unit Files Come From

Most unit files are provided by **RPM packages**.

### Find which package provides a service unit:

```bash
dnf provides */httpd.service
```

**Output:** `httpd-2.4.x-x86_64` provides `/usr/lib/systemd/system/httpd.service`

> 💡 The `*/` wildcard matches any path — useful when you don't know the exact directory.

---

## 3. Checking Service Status — `systemctl status`

```bash
systemctl status httpd
```

### Information shown:

| Field | Meaning |
|-------|---------|
| `Loaded` | Unit file location and whether it's enabled |
| `Active` | Current running state (active/inactive/failed) |
| `Main PID` | Process ID of main service process |
| `Tasks` | Number of processes in the service's control group |
| `Memory` | Memory usage |
| `CPU` | CPU time used |
| `CGroup` | Control group path |
| `Log` | Recent journal entries |

### Key status indicators:

| Indicator | Meaning |
|-----------|---------|
| **Active** | Currently running |
| **Enabled** | Starts automatically at boot |
| **Vendor preset** | Default behavior from RPM package |

> 📌 `systemctl status` uses `less` as a pager. Press `q` to quit.

### Disable pager (returns immediately):

```bash
systemctl --no-pager status httpd
```

---

## 4. Starting and Stopping Services — One-Time Actions

| Action | Command |
|--------|---------|
| Start service now | `systemctl start httpd` |
| Stop service now | `systemctl stop httpd` |
| Restart service (stop + start) | `systemctl restart httpd` |
| Reload configuration (no interruption) | `systemctl reload httpd` |

> 💡 `start`/`stop` affect only the **current session** — not persistent across reboots.

---

## 5. Enabling Services — Persistent Across Reboots

| Action | Command |
|--------|---------|
| Enable (start at boot) | `systemctl enable httpd` |
| Disable (don't start at boot) | `systemctl disable httpd` |

> 💡 `enable`/`disable` affect **boot behavior** — does NOT affect current running state.

---

## 6. Enable + Start Together — `--now`

### Two commands in one:

```bash
systemctl enable --now httpd
```

**What it does:**
- Starts the service **immediately**
- Sets it to start **automatically at boot**

### Combine with `--no-pager`:

```bash
systemctl enable --now httpd --no-pager
```

### Verify:

```bash
systemctl status httpd --no-pager
```

**Expected output:** `Active: active (running)` and `Enabled: enabled`

---

## 7. Reloading vs Restarting — Important Distinction

### The problem:

1. Service starts → loads config file from disk → **into memory**
2. You change config file on disk
3. Service still uses **old config** (in memory)

### Solutions:

| Action | Command | Effect | Disruption? |
|--------|---------|--------|-------------|
| Restart | `systemctl restart httpd` | Stop → Start → loads new config | **Yes** (service downtime) |
| Reload | `systemctl reload httpd` | Sends SIGHUP (signal 1) to reload config | **No** (seamless) |
| Reload or Restart | `systemctl reload-or-restart httpd` | Reloads if supported, otherwise restarts | Minimal |

### How reload works:

- Sends **SIGHUP** (signal 1)
- Service "hangs up" on old config in memory
- "Redials" — reads new config from disk
- **Same PID** — no interruption

> ⚠️ Not all services support `reload`. Use `reload-or-restart` as a safe fallback.

---

## 8. Targets — Grouping Units

### What are targets?

- **Targets** = groups of systemd units
- Similar to **runlevels** in SysV init

### Get default target (boot target):

```bash
systemctl get-default
```

**Example output:** `multi-user.target`

### List dependencies of a target:

```bash
systemctl list-dependencies multi-user.target
```

**Shows:** All units that start when this target starts (including other targets).

### Reverse dependencies (what calls this unit?):

```bash
systemctl list-dependencies --reverse httpd
```

**Output example:**
```
httpd.service
└─multi-user.target
  └─graphical.target
```

> 💡 Color coding: green dot (●) = active, dark/dim = inactive.

---

## 9. Disabling Services — `disable --now`

### Stop service now + prevent boot start:

```bash
systemctl disable --now httpd
```

**What it does:**
- Stops the service **immediately**
- Removes it from being called by `multi-user.target` (or other targets)
- Service will **not** start at boot

### Verify:

```bash
systemctl status httpd --no-pager
```

**Expected:** `Active: inactive (dead)`, `Enabled: disabled`

---

## 10. Masking Services — Preventing ANY Start

### What masking does:

- Prevents service from being started by **systemd OR manually by user**
- Creates a symlink from unit file to `/dev/null`

### Mask a service:

```bash
systemctl mask httpd
```

### Find masked services:

```bash
systemctl list-unit-files --all | grep masked
```

**Example output:** `httpd.service    masked`

### Unmask a service:

```bash
systemctl unmask httpd
```

### Mask vs Disable:

| Action | Current session | Boot behavior | Can user start manually? |
|--------|-----------------|---------------|--------------------------|
| `disable` | No change | Won't start | ✅ Yes (`systemctl start`) |
| `mask` | No change | Won't start | ❌ No (blocked) |

> 💡 Use `mask` for services you **never** want to run (e.g., security policy).

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Check service status (with pager) | `systemctl status httpd` |
| Check service status (no pager) | `systemctl --no-pager status httpd` |
| Start service (one-time) | `systemctl start httpd` |
| Stop service (one-time) | `systemctl stop httpd` |
| Restart service (downtime) | `systemctl restart httpd` |
| Reload config (no downtime) | `systemctl reload httpd` |
| Reload or restart (fallback) | `systemctl reload-or-restart httpd` |
| Enable at boot | `systemctl enable httpd` |
| Disable at boot | `systemctl disable httpd` |
| Enable + start now | `systemctl enable --now httpd` |
| Disable + stop now | `systemctl disable --now httpd` |
| Get default target | `systemctl get-default` |
| List target dependencies | `systemctl list-dependencies multi-user.target` |
| Reverse dependencies | `systemctl list-dependencies --reverse httpd` |
| Mask service (block all starts) | `systemctl mask httpd` |
| Unmask service | `systemctl unmask httpd` |
| List masked services | `systemctl list-unit-files --all \| grep masked` |
| Find package providing unit file | `dnf provides */httpd.service` |

---

# Service States — Quick Reference

| State | Meaning | How to achieve |
|-------|---------|----------------|
| Active + Enabled | Running now + starts at boot | `enable --now` |
| Active + Disabled | Running now but won't start at boot | `start` (but not `enable`) |
| Inactive + Enabled | Not running now but will start at boot | `enable` (but not `start`) |
| Inactive + Disabled | Not running, won't start at boot | `disable` (default for new installs) |
| Masked | Cannot be started at all (blocked) | `mask` |

---

# Reload vs Restart — Visual Comparison

```
┌─────────────────────────────────────────────────────────────┐
│                        RESTART                               │
├─────────────────────────────────────────────────────────────┤
│  Running (old config) → STOP → START → Running (new config) │
│                                                              │
│  • Service downtime (may affect users)                      │
│  • New PID assigned                                          │
│  • Works for all services                                    │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                        RELOAD                                │
├─────────────────────────────────────────────────────────────┤
│  Running (old config) → SIGHUP → Running (new config)       │
│                                                              │
│  • NO downtime (seamless)                                   │
│  • Same PID                                                  │
│  • Only works for services that support SIGHUP              │
└─────────────────────────────────────────────────────────────┘
```

---

# Exam Tip — Validate Your Work

> **Before the timer runs down, reboot your server** to make extra sure services start on boot.

### Validation workflow:

1. Configure service (e.g., `systemctl enable --now httpd`)
2. Verify current status: `systemctl status httpd`
3. **Reboot the system**
4. Reconnect and verify again: `systemctl --no-pager status httpd`
5. If it fails, you have time to troubleshoot

> ⚠️ Don't wait until you have 10 minutes left — rushed troubleshooting leads to mistakes.

---

# Key Takeaways

- **Unit files** come from RPM packages — use `dnf provides */service-name.service` to find which package.
- **`systemctl status`** shows current state, but uses `less` pager — use `--no-pager` to disable.
- **`start`/`stop`** = temporary (current session only).
- **`enable`/`disable`** = persistent (boot behavior only).
- **`enable --now`** = start now + start at boot (best of both).
- **`disable --now`** = stop now + don't start at boot.
- **`reload`** = seamless config update (no downtime, same PID) — not all services support it.
- **`reload-or-restart`** = safe fallback for services that don't support reload.
- **Targets** group units — `multi-user.target` is the default boot target on servers.
- **`list-dependencies`** shows what starts with a target.
- **`list-dependencies --reverse`** shows what calls a unit.
- **`mask`** = prevent service from ever starting (even manually).
- **Exam tip:** Reboot to validate persistent configuration before time runs out.