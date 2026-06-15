# Systemd — Notes

## 1. What Is systemd?

- **Initialization system** and **service manager** for all modern Linux distros
- **Created by Red Hat**
- First process that runs after system boots → **PID 1**
- Starts the rest of the system: background services, login prompts, targets

### What systemd provides:

| Feature | Description |
|---------|-------------|
| Start/stop services | Consistently manages services |
| Dependency management | Handles which services need other services first |
| Monitoring | Tools for logging and troubleshooting |
| Target system | Groups units together (similar to runlevels) |

---

## 2. systemd — PID 1

```bash
ps -f 1
```

or

```bash
systemctl
```

> 💡 Everything traces back to systemd as the ultimate parent process.

---

## 3. Units — The Building Blocks of systemd

systemd divides the system into **units** (manageable components).

### Unit types:

| Unit Type | File Extension | Purpose | Example |
|-----------|---------------|---------|---------|
| **Service** | `.service` | Represents a service/daemon | `sshd.service`, `httpd.service` |
| **Socket** | `.socket` | Associated with a port — starts service when connection made | `foo.socket` on port 8000 → starts `foo.service` |
| **Timer** | `.timer` | Schedule — starts service at specific time | `bar.timer` at 11pm → starts `bar.service` |
| **Path** | `.path` | Monitors file/directory — starts service when accessed | `baz.path` on `/mnt/baz` → starts `baz.service` |
| **Target** | `.target` | Groups units together (like runlevels) | `multi-user.target` |
| Mount | `.mount` | File system mount points | |
| Automount | `.automount` | On-demand mounting | |
| Device | `.device` | Kernel devices | |
| Slice | `.slice` | Control groups (cgroups) — resource management | |
| Scope | `.scope` | Groups processes started externally | |

---

## 4. Viewing Units — `systemctl` Command

### List all active units:

```bash
systemctl
```

or

```bash
systemctl list-units
```

### List units by type:

```bash
systemctl list-units --type=service
```

```bash
systemctl list-units --type=socket
```

```bash
systemctl list-units --type=target
```

```bash
systemctl list-units --type=timer
```

### List **all** units (including inactive):

```bash
systemctl list-units --all
```

### List unit files (not just active units):

```bash
systemctl list-unit-files
```

```bash
systemctl list-unit-files --type=service
```

---

## 5. Unit States — LOAD, ACTIVE, SUB

In `systemctl list-units` output:

| Column | Meaning | Example Values |
|--------|---------|----------------|
| **LOAD** | Unit file loaded correctly? | `loaded`, `not-found`, `error` |
| **ACTIVE** | Is the unit activated? | `active`, `inactive`, `failed`, `activating`, `deactivating` |
| **SUB** | Sub-state (more detail) | `running`, `exited`, `waiting`, `dead`, `failed` |

> 📌 See student guide for complete sub-state reference.

---

## 6. Checking Service Status — `systemctl status`

### Basic command:

```bash
systemctl status sshd
```

> 💡 `.service` extension is **optional** — systemd assumes service unit type.

### What `status` shows:

| Field | Meaning |
|-------|---------|
| Green dot (●) | Unit loaded, active |
| `Loaded` | Location of unit file |
| `Active` | Running state + timestamp |
| `Main PID` | Process ID of main process |
| `Memory/CPU` | Resource usage |
| `CGroup` | Control group path (resource management) |
| `Log` | Recent journal entries for this unit |

### Quick checks:

```bash
systemctl is-active sshd        # Returns: active, inactive, failed
systemctl is-enabled sshd       # Returns: enabled, disabled, static
```

---

## 7. Unit Files — Defining Unit Behavior

### Location of unit files:

| Directory | Purpose | Who manages? |
|-----------|---------|--------------|
| `/usr/lib/systemd/system/` | **Vendor-provided** unit files (from RPMs) | RPM packages |
| `/etc/systemd/system/` | **Local customizations** | System administrator |

> ⚠️ **Do not modify files in `/usr/lib/systemd/system/`** — they are owned by RPM packages and will be replaced on update.

### View a unit file:

```bash
systemctl cat sshd
```

**Output shows:**
- File path (`/usr/lib/systemd/system/sshd.service`)
- Full unit file contents

### Find which RPM provides a unit file:

```bash
dnf provides /usr/lib/systemd/system/httpd.service
```

---

## 8. Target Units — Grouping Units Together

- Targets group one or more units together
- Similar to **runlevels** in SysV init

### Common targets:

| Target | Purpose |
|--------|---------|
| `poweroff.target` | System shutdown |
| `rescue.target` | Single-user mode (maintenance) |
| `multi-user.target` | Normal multi-user operation (no GUI) |
| `graphical.target` | Multi-user with GUI |
| `reboot.target` | System reboot |

### View current default target:

```bash
systemctl get-default
```

---

## 9. Socket Units — On-Demand Service Activation

- Socket unit listens on a port
- When connection is made → systemd starts the matching service

### Example relationship:

| Unit | Purpose |
|------|---------|
| `foo.socket` | Listens on port 8000/tcp |
| `foo.service` | Started when connection hits port 8000 |

> 💡 Saves resources — service only runs when needed.

---

## 10. Timer Units — Scheduled Service Activation

- Timer unit defines a schedule
- When schedule matches → systemd starts the matching service

### Example:

| Unit | Purpose |
|------|---------|
| `bar.timer` | Triggers at 11:00 PM daily |
| `bar.service` | Started at 11:00 PM |

> 💡 Alternative to `cron` — integrated with systemd.

---

## 11. Path Units — Filesystem-Triggered Service Activation

- Path unit monitors a file or directory
- When accessed → systemd starts the matching service

### Example:

| Unit | Purpose |
|------|---------|
| `baz.path` | Monitors `/mnt/baz` |
| `baz.service` | Started when `/mnt/baz` is accessed |

---

## 12. Vendor Preset vs Enabled State

| State | Meaning |
|-------|---------|
| **Vendor preset** | Default behavior set by the RPM package |
| **Enabled** | Unit will start at boot (as configured by admin) |

### Example: SSH daemon

```bash
systemctl list-unit-files --type=service | grep sshd
```

**Output snippet:**
```
sshd.service    enabled    enabled
```

- First `enabled` = current state
- Second `enabled` = vendor preset

### Example: HTTP daemon

```bash
systemctl list-unit-files --type=service | grep httpd
```

**Output snippet:**
```
httpd.service    disabled    disabled
```

> 💡 When you install `httpd` via RPM, the service is **not enabled** by default (vendor preset = disabled). You must enable it manually.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| List all active units | `systemctl` or `systemctl list-units` |
| List units by type | `systemctl list-units --type=service` |
| List all units (including inactive) | `systemctl list-units --all` |
| List unit files | `systemctl list-unit-files` |
| Check service status | `systemctl status sshd` |
| Check if service is active | `systemctl is-active sshd` |
| Check if service is enabled | `systemctl is-enabled sshd` |
| View unit file | `systemctl cat sshd` |
| Get default target | `systemctl get-default` |
| Find RPM providing unit file | `dnf provides /usr/lib/systemd/system/name.service` |

---

# Unit Types Quick Reference

| Unit | File Extension | Trigger | Action |
|------|---------------|---------|--------|
| Service | `.service` | Manual or dependency | Runs service |
| Socket | `.socket` | Network connection | Starts matching service |
| Timer | `.timer` | Time schedule | Starts matching service |
| Path | `.path` | File/directory access | Starts matching service |
| Target | `.target` | Group of units | Starts grouped units |

---

# Unit File Locations

```
/usr/lib/systemd/system/
├── sshd.service          # Vendor-provided (do not edit)
├── httpd.service         # Vendor-provided (do not edit)
└── ...

/etc/systemd/system/
├── multi-user.target.wants/   # Symlinks for enabled services
└── ...                        # Custom overrides (admin-managed)
```

---

# Key Takeaways

- **systemd** = initialization system + service manager (PID 1).
- **Units** = manageable components (services, sockets, timers, paths, targets, etc.).
- **`systemctl`** = primary command to interact with systemd.
- **Service units** (`.service`) = represent daemons/services.
- **Socket units** = start services on-demand when a port is accessed.
- **Timer units** = start services on a schedule (alternative to cron).
- **Path units** = start services when a file/directory is accessed.
- **Target units** = group other units together (like runlevels).
- **Unit files** define unit behavior:
  - `/usr/lib/systemd/system/` = vendor-provided (RPMs) — **do not edit**
  - `/etc/systemd/system/` = local customizations
- **`systemctl status`** shows current state, main PID, resource usage, and recent logs.
- **`systemctl is-active`** and **`is-enabled`** = quick checks.
- Vendor preset = default behavior from RPM. Enabled = administrator-configured boot start.