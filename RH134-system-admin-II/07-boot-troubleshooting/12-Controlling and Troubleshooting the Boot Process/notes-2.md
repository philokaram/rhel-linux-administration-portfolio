# System Initialization — Notes

## 1. The Boot Process — After the Bootloader

```
Bootloader (GRUB2)
        │
        ▼
Kernel + initramfs loaded into memory
        │
        ▼
Kernel initializes hardware (using initramfs drivers)
        │
        ▼
Kernel runs /sbin/init → systemd (PID 1)
        │
        ▼
systemd mounts real root file system
        │
        ▼
systemd pivots to real root (root pivot)
        │
        ▼
systemd reaches default target → system ready
```

---

## 2. initramfs — The Mini Operating System

- **initramfs** = temporary mini OS loaded during boot
- Contains:
  - Kernel modules for hardware needed at boot
  - Initialization scripts
  - A bootable root file system with its own copy of **systemd**

### `initramfs` vs real root:

| Component | initramfs | Real Root |
|-----------|-----------|-----------|
| Location | Memory (temporary) | Disk |
| Purpose | Boot hardware, mount real root | Full system |
| systemd | Yes (temporary) | Yes (permanent) |

---

## 3. `init` → systemd

- `/sbin/init` is a **symbolic link** to `systemd`
- Systemd is **PID 1** — the first process

### Verify:

```bash
ls -l /sbin/init
```

**Output:** `/sbin/init -> /usr/lib/systemd/systemd`

### Override with kernel parameter:

```
init=/bin/bash
```

> 💡 This is used in emergency situations (e.g., resetting root password).

---

## 4. Targets — Grouping Systemd Units

- **Targets** = groups of systemd units
- Targets can call **other targets** (dependencies)

### View default target:

```bash
systemctl get-default
```

### List dependencies of a target:

```bash
systemctl list-dependencies multi-user.target
```

### Grep for targets only:

```bash
systemctl list-dependencies multi-user.target | grep target
```

### List all targets:

```bash
systemctl list-units --type=target --all
```

---

## 5. Common Targets

| Target | Purpose |
|--------|---------|
| `poweroff.target` | System shutdown |
| `rescue.target` | Single-user mode (basic services, writable FS) |
| `emergency.target` | Minimal mode (root read-only, no services) |
| `multi-user.target` | Multi-user, no GUI |
| `graphical.target` | Multi-user with GUI |
| `reboot.target` | System reboot |

---

## 6. Switching Targets — `systemctl isolate`

### Switch to multi-user (non-GUI):

```bash
systemctl isolate multi-user.target
```

### Switch back to graphical (GUI):

```bash
systemctl isolate graphical.target
```

> 💡 `isolate` stops everything not required by the target.

---

## 7. Changing Default Target — `systemctl set-default`

### Set default to multi-user (boot to text mode):

```bash
systemctl set-default multi-user.target
```

### Set default to graphical (boot to GUI):

```bash
systemctl set-default graphical.target
```

> 💡 Changes take effect on the **next boot**.

---

## 8. Rescue and Emergency Targets

### Access via kernel command line:

- Press `E` at GRUB menu
- Add to the `linux` line:

```
systemd.unit=rescue.target
```

or

```
systemd.unit=emergency.target
```

### Comparison:

| Feature | Rescue Target | Emergency Target |
|---------|---------------|------------------|
| Services | Basic services started | Minimal (no services) |
| File system | Writable | **Read-only** |
| Logging | Available | Not available |
| Use case | Fix `/etc/fstab` or dependency loops | Minimal troubleshooting |

### In emergency target (root is read-only):

```bash
mount -o remount,rw /
```

### Continue normal boot:

```bash
exit
```

---

## 9. Rescue/Emergency — Authentication

- Both require **root password**
- If you don't know the root password, you need to reset it first

---

## 10. System Initialization — Visual Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Firmware (UEFI/BIOS) → GRUB2 → Kernel + initramfs       │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Kernel starts → /sbin/init → systemd (PID 1)           │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. systemd mounts real root (from /etc/fstab)              │
│    └── initrd.target → mount /sysroot                       │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Root pivot — switch from initramfs to real root         │
│    └── systemd re-executes from installed binary           │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. systemd reaches default target                           │
│    └── multi-user.target or graphical.target                │
│    └── Starts all dependencies                              │
└─────────────────────────────────────────────────────────────┘
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Get default target | `systemctl get-default` |
| Set default target | `systemctl set-default multi-user.target` |
| List target dependencies | `systemctl list-dependencies multi-user.target` |
| List all targets | `systemctl list-units --type=target --all` |
| Switch target (isolate) | `systemctl isolate multi-user.target` |
| View init link | `ls -l /sbin/init` |
| Remount root as writable | `mount -o remount,rw /` |
| Boot to rescue target | Add `systemd.unit=rescue.target` to kernel command line |
| Boot to emergency target | Add `systemd.unit=emergency.target` to kernel command line |

---

# Target Dependencies — Example

```
graphical.target
        │
        ├── display-manager.service (GNOME/KDE)
        ├── systemd-update-utmp-runlevel.service
        ├── udisks2.service
        │
        └── multi-user.target
                 │
                 ├── atd.service
                 ├── auditd.service
                 ├── cronyd.service
                 ├── sshd.service
                 │
                 └── basic.target
                          │
                          ├── paths.target
                          ├── slices.target
                          ├── sockets.target
                          └── timers.target
```

---

# Key Takeaways

- **initramfs** = temporary mini OS loaded before the real root file system.
- **`/sbin/init`** is a symbolic link to **systemd** (PID 1).
- **Targets** group systemd units (e.g., `multi-user.target`, `graphical.target`).
- **`systemctl isolate`** switches targets on the fly.
- **`systemctl set-default`** changes the boot target permanently.
- **Rescue target** = basic services + writable filesystem (good for troubleshooting).
- **Emergency target** = minimal environment + read-only filesystem (use when rescue target fails).
- **Root password is required** for rescue and emergency targets.
- **Kernel command line** can override the default target using `systemd.unit=...`.
- **Systemd is in full control** — from initramfs through to the final target.