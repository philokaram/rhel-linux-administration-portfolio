# Becoming Another User (Superuser Access) — Notes

## 1. Two Ways to Become Another User

| Command | Stands For | What It Does | Password Required |
|---------|------------|--------------|-------------------|
| `su` | **S**ubstitute **U**ser | Substitute your UID for another user's | **Target user's** password |
| `sudo` | **S**uper **U**ser **DO** | Run a single command as another user | **Your own** password (by default) |

> 💡 The goal isn't just becoming `root` — you can become **any user** you have access to.

---

## 2. The `su` Command — Substitute User

### Syntax:

```bash
su - username
```

| Option | Effect |
|--------|--------|
| `su - username` | Fully assume target user's identity (runs login scripts, sets environment) |
| `su username` | Switch user but **keep current environment** (no login scripts) |
| `su -` (no username) | Assume `root` identity (requires root's password) |

### Example — become `devops` user:

```bash
su - devops
Password: redhat123      # Target user's password
```

**Result:** Prompt changes, environment fully set up as `devops`.

### Verify:

```bash
whoami      # devops
id          # uid=1001(devops) gid=1001(devops)...
```

---

### The Difference — With Dash (`-`) vs Without Dash

| With `su -` | Without `su` |
|-------------|---------------|
| Processes login scripts | Does NOT process login scripts |
| Full environment setup | Partial/inherited environment |
| `pwd` → target user's home | `pwd` → previous directory |
| `echo $USER` → target user | `echo $USER` → original user |
| **Recommended** | Use with caution |

### Example without dash:

```bash
su root                    # No dash
Password: (root's password)
pwd                        # Still /home/student (not /root)
echo $USER                 # student (not root!)
id                         # uid=0(root) gid=0(root) ← actually root!
```

> ⚠️ **Important:** You are technically root (`uid=0`) but environment is mixed — can cause confusion.

### RHEL 10 Note:

- **Root account is locked by default**
- `su -` (to root) will **fail** unless you unlock root first
- **Better approach:** Use `sudo`

---

## 3. The `sudo` Command — Run Commands as Another User

### Default behavior:

- Run command as **root** (if no user specified)
- Prompt for **your own** password
- Password cached for **5 minutes** (default)

### Syntax:

```bash
sudo command
```

### Example — view protected log file:

```bash
sudo tail -10 /var/log/messages
```

**Process:**
1. Prompts for **student's** password
2. Validates user is allowed to use sudo
3. Runs `tail -10 /var/log/messages` as root

> 💡 You don't need root's password — just your own (if authorized).

### Run command as another user (not root):

```bash
sudo -u username command
```

**Example:**
```bash
sudo -u devops whoami
```

---

## 4. The `sudo` Configuration — `/etc/sudoers`

### Important rules:

| Rule | Why |
|------|-----|
| **Never edit `/etc/sudoers` directly** | Syntax errors can lock you out of sudo permanently |
| **Always use `visudo`** | Validates syntax before saving |
| **Use drop-in directory** `/etc/sudoers.d/` | Better organization, easier management |

### Default sudo rule (for `wheel` group):

```bash
grep wheel /etc/sudoers
```

**Output (not commented):**
```
%wheel ALL=(ALL) ALL
```

| Part | Meaning |
|------|---------|
| `%wheel` | Group `wheel` (members) |
| `ALL=` | From any host |
| `(ALL)` | Can run commands as any user |
| `ALL` | Can run any command |

> 💡 By default, members of the `wheel` group can use `sudo` and must enter their **own password**.

### Passwordless sudo (commented out by default):

```
%wheel ALL=(ALL) NOPASSWD: ALL
```

> Remove the `#` to enable passwordless sudo for wheel group.

---

## 5. Best Practice — Drop-in Directory

### Location:

```bash
/etc/sudoers.d/
```

### Create a custom sudo rule:

```bash
sudo visudo -f /etc/sudoers.d/student
```

### Example rule (passwordless for `student` group):

```
%students ALL=(ALL) NOPASSWD: ALL
```

| Part | Meaning |
|------|---------|
| `%students` | Members of group `students` |
| `ALL=` | From any host |
| `(ALL)` | Can run as any user |
| `NOPASSWD:` | No password prompt |
| `ALL` | Can run any command |

### Verify the rule works:

```bash
sudo -k           # Clear cached password
sudo whoami       # root (no password prompt!)
```

---

## 6. Useful `sudo` Commands

| Command | Purpose |
|---------|---------|
| `sudo -i` | Get interactive root shell (full login environment) |
| `sudo -s` | Get root shell (current environment, no login scripts) |
| `sudo -u user` | Run command as specific user |
| `sudo -k` | Clear cached password (force re-authentication) |
| `sudo -l` | List current user's sudo privileges |
| `visudo -f /path/to/file` | Edit sudoers file with syntax validation |

---

## 7. Security — Failed `sudo` Attempts Are Logged

When a user tries to use `sudo` without permission:

```bash
sudo tail /var/log/secure
```

**Example entry:**
```
Aug 26 10:00:00 servera sudo[12345]: student : user NOT in sudoers ; TTY=pts/0 ; PWD=/home/student ; USER=root ; COMMAND=/bin/ls
```

> 🔒 All failed sudo attempts are logged — important for security auditing.

---

# Command Reference Table

| Task | Command |
|------|---------|
| Switch to another user (full env) | `su - username` |
| Switch to root (if allowed) | `su -` |
| Run command as root (sudo) | `sudo command` |
| Run command as another user | `sudo -u username command` |
| Get interactive root shell | `sudo -i` |
| Clear sudo password cache | `sudo -k` |
| List sudo privileges | `sudo -l` |
| Edit sudoers safely | `visudo` |
| Edit sudoers drop-in file | `visudo -f /etc/sudoers.d/filename` |
| View failed sudo attempts | `sudo tail /var/log/secure` |

---

# Comparison: `su` vs `sudo`

| Feature | `su` | `sudo` |
|---------|------|--------|
| Requires target user's password | ✅ | ❌ |
| Requires your own password | ❌ | ✅ (default) |
| Can run single command | ❌ (switches user) | ✅ |
| Provides interactive shell | ✅ | ✅ (`sudo -i`) |
| Logs all attempts | ❌ | ✅ (to `/var/log/secure`) |
| Fine-grained command control | ❌ | ✅ |
| Preferred for day-to-day admin | ❌ | ✅ |

---

# Best Practices Summary

| Practice | Why |
|----------|-----|
| Use `sudo` instead of `su` | Better logging, no need for root password |
| Use `sudo -i` for interactive root shell | Full environment, proper login |
| Never edit `/etc/sudoers` directly | Syntax errors = permanent lockout |
| Use `visudo` for all sudoers edits | Validates syntax before saving |
| Use drop-in directory `/etc/sudoers.d/` | Easier management, upgrade-safe |
| Use `NOPASSWD:` sparingly | Security trade-off for convenience |
| Clear cache with `sudo -k` when done | Prevents unauthorized use of cached credentials |

---

# Key Takeaways

- **`su`** = **S**ubstitute **U**ser — requires **target user's password**. Use `su -` for full environment.
- **`sudo`** = **S**uper **U**ser **DO** — requires **your own password** (by default). Preferred for day-to-day admin.
- **Root's power** comes from UID 0, but root account may be **locked by default** in RHEL 10.
- **`/etc/sudoers`** controls sudo access — **never edit directly**, always use `visudo`.
- **Drop-in directory** `/etc/sudoers.d/` = best practice for custom rules.
- **`%wheel ALL=(ALL) ALL`** = default rule allowing wheel group members to use sudo.
- **`NOPASSWD:`** = passwordless sudo (convenience vs security trade-off).
- **Failed sudo attempts** are logged in `/var/log/secure` — great for auditing.
- **`sudo -i`** = best way to get interactive root shell (full login environment).
- **`sudo -k`** = clear cached password (force re-auth on next sudo).