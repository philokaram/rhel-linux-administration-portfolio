# The Bootloader and Kernel Command Line — Notes

## 1. The Boot Process

```
Power On
    │
    ▼
Firmware (UEFI or BIOS)
    │
    ▼
GRUB2 (Bootloader)
    │
    ▼
Linux Kernel
    │
    ▼
Login Screen
```

### Firmware types:

| Type | Description |
|------|-------------|
| **UEFI** | Modern firmware (uses EFI System Partition at `/boot/efi`) |
| **BIOS** | Legacy firmware (uses Master Boot Record) |

---

## 2. GRUB2 — The Bootloader

- **GRUB2** = **G**rand **U**nified **B**ootloader (version 2)
- It decides:
  - Which kernel to boot
  - What options to pass to the kernel
  - How the system should boot

### GRUB2 menu:

- Press **`E`** at the GRUB menu to edit the boot entry
- Find the line starting with `linux` — this is the **kernel command line**
- Everything after the kernel path = **kernel arguments**

---

## 3. Kernel Command Line — Arguments

Kernel arguments influence how the kernel initializes.

### Common arguments:

| Argument | Purpose |
|----------|---------|
| `quiet` | Suppress boot messages (less verbose) |
| `loglevel=3` | Set kernel log level (3 = errors only) |
| `rd.break` | Break to rescue shell before boot |
| `init=/bin/bash` | Boot directly into bash (emergency) |
| `selinux=0` | Disable SELinux (not recommended) |
| `video=640x480` | Set console resolution |
| `console=ttyS0` | Use serial console |

> 💡 You can **add** or **remove** kernel arguments temporarily by pressing `E` at the GRUB menu.

---

## 4. GRUBBY — The Bootloader Management Tool

- **`grubby`** = command-line tool to manage GRUB2 configuration
- **Safe and consistent** — better than manually editing GRUB2 config files
- Works across **one or more kernels**

### Key commands:

```bash
grubby --default-kernel          # Show default kernel
grubby --default-index            # Show default kernel index
grubby --info=ALL                 # Show info for all kernels
grubby --info=0                   # Show info for kernel index 0
grubby --set-default-index=1      # Set default kernel to index 1
```

---

## 5. Viewing Kernel Information — `grubby --info=ALL`

```bash
grubby --info=ALL | less
```

**Example output for one kernel:**
```
index=0
kernel=/boot/vmlinuz-6.1.0-0.el10.x86_64
args="ro root=/dev/mapper/rhel-root quiet loglevel=3"
root=/dev/mapper/rhel-root
initrd=/boot/initramfs-6.1.0-0.el10.x86_64.img
title="Red Hat Enterprise Linux 10"
id=6.1.0-0.el10.x86_64
```

---

## 6. Adding Kernel Arguments — `grubby --add-args`

### Add arguments to all kernels:

```bash
grubby --update-kernel ALL --add-args="quiet loglevel=3"
```

### Verify:

```bash
grubby --info=ALL | grep args
```

### Remove specific arguments from all kernels:

```bash
grubby --update-kernel ALL --remove-args="quiet"
```

### Verify:

```bash
grubby --info=ALL | grep args
```

---

## 7. Changing the Default Kernel — `grubby --set-default-index`

### List all kernels:

```bash
grubby --info=ALL | grep title
```

### Set default kernel:

```bash
grubby --set-default-index=1
```

### Verify:

```bash
grubby --default-index
```

---

## 8. Getting Default Kernel Info

### Using command substitution:

```bash
grubby --info=$(grubby --default-kernel)
```

**What it does:**
1. `grubby --default-kernel` returns the default kernel path
2. `$(...)` substitutes that value
3. `grubby --info=` shows the info for that kernel

> 💡 Useful for examining the current default kernel in scripts.

---

## 9. Why Use `grubby`? — Benefits

| Benefit | Description |
|---------|-------------|
| **Safety** | Avoids manual errors editing GRUB2 config |
| **Consistency** | Updates all specified kernels |
| **Scriptable** | Works in automation scripts |
| **Persistent** | Changes survive kernel updates |

> 💡 **Red Hat support** often provides kernel arguments to troubleshoot issues. `grubby` lets you apply them safely.

---

## 10. GRUBBY — Quick Reference

| Task | Command |
|------|---------|
| Show default kernel | `grubby --default-kernel` |
| Show default kernel index | `grubby --default-index` |
| Show all kernel info | `grubby --info=ALL` |
| Show info for specific kernel | `grubby --info=0` |
| Set default kernel by index | `grubby --set-default-index=1` |
| Add args to all kernels | `grubby --update-kernel ALL --add-args="quiet loglevel=3"` |
| Remove args from all kernels | `grubby --update-kernel ALL --remove-args="quiet"` |
| Add args to current kernel | `grubby --update-kernel DEFAULT --add-args="crashkernel=auto"` |

---

# Kernel Command Line — Common Arguments

| Argument | Effect |
|----------|--------|
| `quiet` | Suppress boot messages |
| `loglevel=3` | Only show errors (KERN_ERR and above) |
| `loglevel=7` | Show all messages (debug) |
| `rd.break` | Break to emergency shell |
| `init=/bin/bash` | Boot directly to bash |
| `selinux=0` | Disable SELinux (not recommended) |
| `selinux=1` | Enable SELinux (default) |
| `video=640x480` | Console resolution |
| `console=ttyS0` | Serial console |
| `rhgb` | Red Hat graphical boot |

---

# Boot Process — GRUB2 Interaction

```
┌─────────────────────────────────────────────────────────────┐
│ 1. GRUB2 Menu Appears                                       │
│    └── Press E to edit the boot entry                      │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Locate the "linux" line                                  │
│    └── This is the kernel command line                     │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Add or remove kernel arguments                           │
│    └── e.g., remove "quiet", add "rd.break"                │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Press Ctrl+X to boot with modified arguments            │
│    └── This is a ONE-TIME change (temporary)              │
└─────────────────────────────────────────────────────────────┘
```

---

# Key Takeaways

- **GRUB2** is the bootloader on RHEL (Grand Unified Bootloader).
- **Kernel command line** arguments influence how the kernel initializes.
- **Press `E`** at the GRUB menu to temporarily edit kernel arguments.
- **`grubby`** is the safe, consistent way to manage kernel arguments permanently.
- **`grubby --info=ALL`** shows all kernels and their arguments.
- **`grubby --update-kernel ALL --add-args`** adds arguments to all kernels.
- **`grubby --update-kernel ALL --remove-args`** removes arguments from all kernels.
- **`grubby --set-default-index`** changes the default boot kernel.
- **Do not manually edit** GRUB2 configuration files — use `grubby`.
- **`grubby`** is scriptable and recommended by Red Hat support.