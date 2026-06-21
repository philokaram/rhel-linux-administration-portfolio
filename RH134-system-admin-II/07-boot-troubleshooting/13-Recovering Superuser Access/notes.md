# Regaining Superuser Access — Notes

## 1. The Big Picture

- **Regaining root access is a feature, not a flaw** — RHEL is designed to let admins recover control
- **Physical security matters** — if someone has console access, they can attempt these same recovery steps
- **Secure GRUB with a password** to prevent unauthorized recovery attempts

---

## 2. Method 1 — Rescue Mode from Installation Media (Cleanest Approach)

### Use case:

- System is fully installed but root password is lost
- You have access to the RHEL installation ISO/Media

### Steps:

1. **Boot from RHEL installation media**

2. Select **Rescue an installed system**

3. Choose **Continue** (menu item 1)

4. System is mounted at `/mnt/sysroot`

5. **chroot** into your system:

```bash
chroot /mnt/sysroot
```

6. **Change the root password:**

```bash
passwd root
```

7. **Trigger SELinux relabel** (required because SELinux is disabled in rescue mode):

```bash
touch /.autorelabel
```

8. **Reboot** the system:

```bash
reboot
```

9. System will:
   - Perform full SELinux relabel on next boot
   - Reboot once more
   - Come up with the new root password

---

## 3. Why `/.autorelabel` Is Required

| Environment | SELinux Status | Issue |
|-------------|----------------|-------|
| Rescue mode | Disabled | Changes to `/etc/shadow` may not have correct SELinux labels |
| Normal boot | Enforcing | Without correct labels, password change may be ignored |

> 💡 `touch /.autorelabel` ensures a full SELinux relabel on the next boot, fixing labels on changed files.

---

## 4. Method 2 — Interrupting the Boot Process (Emergency Use)

### Use case:

- No access to installation media
- Emergency only — riskier than rescue mode

### Steps:

1. **Interrupt the boot process** (press `E` at GRUB menu)

2. **Find the line starting with `linux`** (kernel command line)

3. **Add `init=/bin/bash`** to the end of the line

4. **Press `Ctrl+X`** to boot

5. **Remount root as read-write:**

```bash
mount -o remount,rw /
```

6. **Change the root password:**

```bash
passwd root
```

7. **Trigger SELinux relabel:**

```bash
touch /.autorelabel
```

8. **Power off or reboot:**

```bash
poweroff
```

or

```bash
reboot
```

> ⚠️ This method bypasses the normal boot process. Only use in emergencies when you can't access rescue media.

---

## 5. Method 3 — Fixing Console Resolution Issues

### Problem:

- Console may not attach to the correct tty
- Resolution may be too low/high to work with

### Persistent fix — `grubby`:

**Remove console arguments from all kernels:**

```bash
grubby --update-kernel ALL --remove-args console
```

**Add video resolution:**

```bash
grubby --update-kernel ALL --args video=640x480
```

### Interactive fix (at GRUB):

1. Press `E` at GRUB menu
2. Find the `linux` line
3. Remove `console=ttyS0` entries
4. Add `video=640x480`
5. Add `init=/bin/bash`
6. Press `Ctrl+X`

---

## 6. Securing GRUB — Preventing Unauthorized Recovery

### Set GRUB password:

```bash
grub2-setpassword
```

**Enter password:** `secret123`

### Result:

- Editing GRUB menu entries now requires **username + password**
- Username: `root`
- Password: whatever you set

> 💡 This prevents unauthorized users from using the boot interruption method.

---

## 7. SELinux Relabel — `chcon` vs `semanage`

| Action | Persistence | After `/.autorelabel` |
|--------|-------------|----------------------|
| `chcon -t type /path` | ❌ Not persistent | **Wiped** |
| `semanage fcontext -a -t type "/path(/.*)?"` | ✅ Persistent | **Survives** |

> 💡 **Best practice:** Use `semanage` for permanent context changes. `chcon` changes are lost after a full relabel.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Enter rescue mode | Boot from installation media → Rescue an installed system |
| chroot into system | `chroot /mnt/sysroot` |
| Change root password | `passwd root` |
| Trigger SELinux relabel | `touch /.autorelabel` |
| Reboot | `reboot` |
| Interrupt boot | Press `E` at GRUB menu |
| Add emergency shell | `init=/bin/bash` on kernel command line |
| Remount root read-write | `mount -o remount,rw /` |
| Remove console args (persistent) | `grubby --update-kernel ALL --remove-args console` |
| Add video resolution (persistent) | `grubby --update-kernel ALL --args video=640x480` |
| Set GRUB password | `grub2-setpassword` |
| Power off | `poweroff` |

---

# Recovery Methods — Comparison

| Method | Difficulty | Cleanliness | Use Case |
|--------|------------|-------------|----------|
| Rescue mode (ISO) | Low | High | Preferred — use when you have installation media |
| Boot interruption (`init=/bin/bash`) | Medium | Low | Emergency — no installation media available |
| GRUB password | Low | N/A | **Prevent** unauthorized recovery |

---

# Security Considerations

```
Physical Access = System Access
        │
        ├── You → Legitimate recovery
        │
        └── Attacker → Can bypass security

Solutions:
├── Secure GRUB with password
├── Use LUKS disk encryption
├── Secure physical access to servers
└── Use strong root password
```

> 💡 **Physical security is your first defense.** If someone has console access, they can potentially recover your system.

---

# Key Takeaways

- **Recovering root access is a feature** — RHEL provides legitimate ways to regain access.
- **Physical security matters** — console access means someone can attempt recovery.
- **Rescue mode (from installation media)** is the **cleanest and most supported method**.
- **Always trigger SELinux relabel** (`touch /.autorelabel`) after changing root password in rescue mode.
- **Emergency method:** Interrupt boot with `E` → add `init=/bin/bash` → remount root → change password.
- **Secure GRUB** with `grub2-setpassword` to prevent unauthorized recovery attempts.
- **Use `semanage fcontext`** for persistent SELinux context changes — `chcon` changes are lost on relabel.
- **Practice until muscle memory kicks in** — this skill is crucial for real-world administration.
- **Exam tip:** Know both methods (rescue mode and boot interruption). Rescue mode is preferred when available.