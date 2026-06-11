# Special Permissions & umask — Notes

## 1. The "I Lied" Moment

Previously: "There are only 3 permissions" — that was a simplification.

**Actually:** There are **6 permissions** (3 regular + 3 special)

| Type | Permissions |
|------|-------------|
| Regular | Read (`r`), Write (`w`), Execute (`x`) |
| Special | Set UID (`s`), Set GID (`s`), Sticky Bit (`t`) |

---

## 2. Set User ID (Set UID / SUID)

### What it does:

> A file executes with the **permissions of the file's owner**, NOT the user running it.

### Where it applies:

- **Executable files only** (not directories, not scripts)

### Octal value: **4** (prepended to regular permissions)

### Example: `chmod 4755` = Set UID + `rwx r-x r-x`

### Visual indicator:

```bash
ls -l /usr/bin/tac
# -rwsr-xr-x 1 root root ... /usr/bin/tac
#    ^ -- 's' instead of 'x' for owning user
```

| Display | Meaning |
|---------|---------|
| `rws` | Set UID + execute permission |
| `rwS` | Set UID **without** execute (invalid — don't use) |

---

## 3. SUID in Action — The `tac` Example

### Background:

| File | Student can read? |
|------|-------------------|
| `/etc/passwd` | ✅ Yes (world-readable) |
| `/etc/shadow` | ❌ No (only root) |

### Without SUID:

```bash
cat /etc/shadow    # Permission denied
tac /etc/shadow    # Permission denied (same as cat)
```

### With SUID on `tac` (as root):

```bash
chmod 4755 /usr/bin/tac
ls -l /usr/bin/tac
# -rwsr-xr-x 1 root root ... /usr/bin/tac
```

### Now as `student`:

```bash
cat /etc/shadow    # Permission denied
tac /etc/shadow    # ✅ WORKS! (runs as root)
```

> ⚠️ **Security implication:** SUID binaries are powerful and dangerous. Only trusted executables should have SUID.

---

## 4. Set Group ID (SGID)

### On **Executable Files**:

> File executes with permissions of the **file's group**, NOT the user's group.

### On **Directories**:

> Newly created files inside inherit the **directory's group** as their owning group.

### Octal value: **2** (prepended to regular permissions)

### Example directory SGID:

```bash
chmod 2770 /data
# drwxrws--- 2 root developers ... /data
#        ^ -- 's' instead of 'x' for group
```

---

## 5. SGID on Directories — Demonstration

### Without SGID:

```bash
ls -ld /data
# drwxrwx--- 2 root developers ... /data

su - mo
cd /data
touch mo.1
ls -l mo.1
# -rw-rw-r-- 1 mo mo ... mo.1   ← Group = mo's primary group (mo)
```

### With SGID:

```bash
chmod g+s /data
# or chmod 2770 /data

ls -ld /data
# drwxrws--- 2 root developers ... /data

touch mo.2
ls -l mo.2
# -rw-rw-r-- 1 mo developers ... mo.2   ← Group = directory's group (developers)
```

> 💡 SGID is great for **collaborative directories** — all files share the same group.

---

## 6. Sticky Bit

### What it does:

> Only the **owner** of a file can delete it from a directory.

### Where it applies:

- **Directories only** (not files)

### Octal value: **1** (prepended to regular permissions)

### Example:

```bash
chmod 1777 /tmp
ls -ld /tmp
# drwxrwxrwt 2 root root ... /tmp
#            ^ -- 't' instead of 'x' for other
```

| Display | Meaning |
|---------|---------|
| `rwt` | Sticky bit + execute permission |
| `rwT` | Sticky bit **without** execute (invalid) |

---

## 7. Sticky Bit in Action

### Without sticky bit:

```bash
# alisson (member of developers) can delete mo's file
rm /data/mo.1   # ✅ Works (has write on directory)
```

### With sticky bit:

```bash
chmod 1770 /data   # SGID + Sticky
# or chmod o+t /data

ls -ld /data
# drwxrwx--T 2 root developers ... /data
#            ^ uppercase T = sticky bit, no execute for other

# alisson tries to delete mo.2
rm /data/mo.2
# Operation not permitted  ❌
```

> 💡 Sticky bit is great for **shared directories** like `/tmp` — users can't delete each other's files.

---

## 8. Combining Special Permissions — Octal Values

| Special Permission | Octal Value |
|--------------------|-------------|
| Set UID (SUID) | 4 |
| Set GID (SGID) | 2 |
| Sticky Bit | 1 |

### Combined octal syntax: `chmod XYZW file`

Where `X` = special permissions, `Y` = user, `Z` = group, `W` = other

| Command | Special Bits | Regular Permissions |
|---------|--------------|---------------------|
| `chmod 4755` | SUID (4) | `755` (rwx r-x r-x) |
| `chmod 2770` | SGID (2) | `770` (rwx rwx ---) |
| `chmod 1777` | Sticky (1) | `777` (rwx rwx rwx) |
| `chmod 3770` | SGID + Sticky (2+1=3) | `770` (rwx rwx ---) |
| `chmod 6755` | SUID + SGID (4+2=6) | `755` (rwx r-x r-x) |

---

## 9. Default Permissions — Kernel Defaults

### Linux kernel defaults (before umask):

| File Type | Default Permissions |
|-----------|---------------------|
| **Directories** | `777` (rwxrwxrwx) |
| **Files** | `666` (rw-rw-rw-) |

> ⚠️ These are **kernel defaults**, NOT RHEL defaults. RHEL uses a **umask** to restrict them.

---

## 10. umask — Restricting Default Permissions

### What is umask?

A **mask** that removes permissions from the kernel defaults.

### Formula:

```
Final permissions = Default permissions - umask
```

### Common umask values:

| umask | Directory Result | File Result |
|-------|------------------|-------------|
| `022` | `755` (rwx r-x r-x) | `644` (rw- r-- r--) |
| `002` | `775` (rwx rwx r-x) | `664` (rw- rw- r--) |
| `077` | `700` (rwx --- ---) | `600` (rw- --- ---) |

### Check current umask:

```bash
umask
# 0022
```

### Set umask (temporarily):

```bash
umask 077
mkdir testdir
touch testfile
ls -ld testdir testfile
# drwx------ 2 student student ... testdir
# -rw------- 1 student student ... testfile
```

---

## 11. RHEL Default umask — `/etc/login.defs`

### System-wide umask setting:

```bash
grep -i umask /etc/login.defs
# UMASK 022
```

### Where umask is set:

| Location | Scope |
|----------|-------|
| `/etc/login.defs` | System-wide (all users) |
| `~/.bashrc` | Per-user (login scripts) |
| `~/.profile` | Per-user (alternative) |

> 💡 **RHEL default = 022** → directories: `755`, files: `644`

---

## 12. Special Permissions — Visual Summary

### SUID on file:

```bash
-rwsr-xr-x   # Lowercase s = SUID + execute
-rwSr--r--   # Uppercase S = SUID + NO execute (broken)
```

### SGID on directory:

```bash
drwxrws---   # Lowercase s = SGID + execute
drwxrwS---   # Uppercase S = SGID + NO execute (broken)
```

### Sticky bit on directory:

```bash
drwxrwxrwt   # Lowercase t = Sticky + execute
drwxrwxrwT   # Uppercase T = Sticky + NO execute (common on /tmp?)
```

> Actually `/tmp` is `drwxrwxrwt` — it has execute permission for other.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Set SUID (octal) | `chmod 4755 /usr/bin/tac` |
| Set SGID on dir (octal) | `chmod 2770 /data` |
| Set Sticky bit (octal) | `chmod 1777 /tmp` |
| Set SUID+SGID (octal) | `chmod 6755 executable` |
| Set SGID+Sticky (octal) | `chmod 3770 /data` |
| Set SUID (symbolic) | `chmod u+s /usr/bin/tac` |
| Set SGID (symbolic) | `chmod g+s /data` |
| Set Sticky (symbolic) | `chmod o+t /data` |
| Remove SUID | `chmod u-s /usr/bin/tac` |
| Remove SGID | `chmod g-s /data` |
| Remove Sticky | `chmod o-t /data` |
| View umask | `umask` |
| Set umask (temp) | `umask 077` |
| View system umask | `grep UMASK /etc/login.defs` |

---

# Special Permissions Cheat Sheet

| Permission | Octal | Applies to | Effect |
|------------|-------|------------|--------|
| **SUID** | 4 | Executable files | Runs as file owner, not user |
| **SGID** | 2 | Files + Directories | File: runs as file group / Dir: new files inherit group |
| **Sticky** | 1 | Directories only | Only file owners can delete |

---

# umask Calculation Examples

### Default: `777` (dirs) or `666` (files) MINUS umask

| umask | Directory Calculation | Directory Result | File Calculation | File Result |
|-------|----------------------|------------------|------------------|-------------|
| `022` | `777 - 022` | `755` (rwx r-x r-x) | `666 - 022` | `644` (rw- r-- r--) |
| `002` | `777 - 002` | `775` (rwx rwx r-x) | `666 - 002` | `664` (rw- rw- r--) |
| `077` | `777 - 077` | `700` (rwx --- ---) | `666 - 077` | `600` (rw- --- ---) |
| `000` | `777 - 000` | `777` (rwx rwx rwx) | `666 - 000` | `666` (rw- rw- rw-) |

---

# Key Takeaways

- **SUID (4)** = runs as file owner (executable files only)
- **SGID (2)** = file: runs as file group / directory: new files inherit group
- **Sticky bit (1)** = only file owners can delete (directories only)
- **Visual indicators:** lowercase = permission + special bit; uppercase = special bit only (broken)
- **Combined octal:** `chmod 3770` = SGID + Sticky + `770`
- **Kernel defaults:** `777` for dirs, `666` for files
- **umask subtracts** from kernel defaults
- **RHEL default umask = 022** → dirs `755`, files `644`
- **SUID is dangerous** — only set on trusted executables
- **SGID + Sticky** = perfect for collaborative directories
- `/tmp` has `1777` = sticky bit prevents users from deleting each other's temp files