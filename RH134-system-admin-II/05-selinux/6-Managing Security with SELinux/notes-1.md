# Operating SELinux — Notes

## 1. Ability vs Authority — The Analogy

- **Ability** = You physically can do something (e.g., empty a water bottle on equipment)
- **Authority** = You are allowed to do it (SELinux enforces this)

| Security Type | Analogy |
|---------------|---------|
| Discretionary Access Control (DAC) | You decide if it's a good idea |
| Mandatory Access Control (MAC) | Police at every intersection — rules cannot be broken |

> 💡 SELinux = **Mandatory Access Control** on top of DAC. Rules are enforced, not optional.

---

## 2. SELinux Modes of Operation

| Mode | Behavior | Analogy |
|------|----------|---------|
| **Enforcing** | Blocks violations, logs denials | Police actively stopping cars |
| **Permissive** | Logs violations but does NOT block | Police writing tickets, but you keep driving |
| **Disabled** | No SELinux checks at all | No police, no security guards |

> 💡 **Enforcing** is the default and recommended mode on RHEL.

### View current mode:

```bash
getenforce
```

### Temporary mode change:

```bash
setenforce 0        # Permissive
setenforce 1        # Enforcing
```

> ⚠️ **Best practice:** Don't set the entire system to Permissive. If troubleshooting, set only the specific subsystem to Permissive.

---

## 3. Persistent SELinux Configuration — `/etc/selinux/config`

### View configuration (without comments):

```bash
decomment /etc/selinux/config
```

**Output:**
```
SELINUX=enforcing
SELINUXTYPE=targeted
```

| Directive | Meaning |
|-----------|---------|
| `SELINUX` | Mode at boot: `enforcing`, `permissive`, `disabled` |
| `SELINUXTYPE` | Policy type: `targeted` (default), `mls` (multi-level security) |

> 💡 **Targeted policy:** Only high-risk services are confined (like security guards at jewelry stores, not every shop).

---

## 4. Disabling SELinux — Kernel Parameters

### Why disabling SELinux is not recommended:

- It disables all SELinux protection
- Switching from `disabled` to `enforcing` requires a relabel (can be complex)
- **Don't disable SELinux** — learn to work with it

### Check kernel command line:

```bash
cat /proc/cmdline
```

**Look for:** `selinux=0` (disables SELinux)

### Remove `selinux=0` kernel parameter:

```bash
grubby --update-kernel ALL --remove-args selinux
```

> 💡 This restores SELinux based on `/etc/selinux/config`.

### After changing SELinux mode:

- If booted in `permissive`, then `setenforce 1` → **reboot recommended**
- Existing processes may not have full protection

---

## 5. SELinux Labels — The 5 Contexts

Everything in SELinux has a **label**:

```
user_u:role_r:type_t:sensitivity:c0.c1023
   │       │       │       │       │
   │       │       │       │       └── Category
   │       │       │       └────────── Sensitivity
   │       │       └────────────────── Type (what we focus on in this course)
   │       └────────────────────────── Role
   └────────────────────────────────── User
```

> 💡 In a basic course, we focus on the **type context** (ending in `_t`).

---

## 6. Viewing SELinux Labels

### User context:

```bash
id -Z
```

**Output:** `unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023`

### Process context:

```bash
ps -Z
```

### File/directory context:

```bash
ls -ldZ /etc
```

**Output:** `system_u:object_r:etc_t:s0 /etc`

---

## 7. Troubleshooting with Permissive Mode

### Workflow:

1. Suspect SELinux is blocking something
2. Set **specific subsystem** to Permissive (or entire system for testing)
3. Test if the transaction works
4. If it works → adjust the label (fix the context)
5. Return to Enforcing mode

> ⚠️ **Don't leave the system in Permissive mode** — it reduces security.

### Temporary global Permissive mode (for testing only):

```bash
setenforce 0
# test your application
setenforce 1
```

> 💡 Better: Use `semanage permissive -a <domain>` to set a specific domain to Permissive.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View current mode | `getenforce` |
| Set to Permissive (temporary) | `setenforce 0` |
| Set to Enforcing (temporary) | `setenforce 1` |
| View persistent config | `cat /etc/selinux/config` |
| View kernel command line | `cat /proc/cmdline` |
| Remove SELinux kernel param | `grubby --update-kernel ALL --remove-args selinux` |
| View user SELinux context | `id -Z` |
| View process SELinux context | `ps -Z` |
| View file SELinux context | `ls -Z` |
| View directory context | `ls -ldZ /etc` |

---

# SELinux Modes — Quick Reference

| Mode | `getenforce` | `setenforce` | Behavior |
|------|-------------|--------------|----------|
| Enforcing | `Enforcing` | `1` | Blocks violations, logs |
| Permissive | `Permissive` | `0` | Only logs, doesn't block |
| Disabled | `Disabled` | (N/A) | No checks, no labels |

---

# Persistent Configuration — Mode vs Policy

```
/etc/selinux/config
        │
        ├── SELINUX=enforcing     # Mode at boot
        │    └── enforcing, permissive, disabled
        │
        └── SELINUXTYPE=targeted  # Policy type
             └── targeted (default), mls (extreme security)
```

---

# Key Takeaways

- **SELinux** = Mandatory Access Control (MAC) on top of traditional DAC.
- **Enforcing mode** = actively blocks violations (default, recommended).
- **Permissive mode** = logs violations but does NOT block (good for troubleshooting).
- **Disabled mode** = SELinux completely off — not recommended.
- **Use `getenforce`** to check current mode.
- **Use `setenforce`** for temporary mode changes.
- **Persistent configuration** is in `/etc/selinux/config`.
- **Targeted policy** = only high-risk services are confined.
- **Labels** = `user:role:type:sensitivity:category` (we focus on **type** `_t`).
- **`ls -Z`** shows SELinux contexts on files.
- **`ps -Z`** shows SELinux contexts on processes.
- **`id -Z`** shows your user's SELinux context.
- **Disabling SELinux via kernel param** (`selinux=0`) is not recommended — learn to work with it instead.
- **Reboot recommended** after changing from `permissive` to `enforcing` to ensure all processes get full protection.