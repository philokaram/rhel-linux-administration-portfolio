# Linux Command Line (Bash) — Notes

## 1. What is the Command Line?

* A **text-based interface** used to interact with Linux.
* Commands are interpreted by a program called a **shell**.
* The default shell in RHEL is **Bash (Bourne Again Shell)**.
* Bash is powerful and supports scripting and automation.

---

## 2. Shell Prompt

* Shows the system is ready for input.

### Types:

* **Normal user prompt:**

  ```
  user@host:~$
  ```
* **Root (superuser) prompt:**

  ```
  root@host:~#
  ```
* `$` = normal user
* `#` = root user (important for safety)

---

## 3. Structure of a Linux Command

Basic format:

```
command [options] [arguments]
```

### Example:

```
du -sh /home/student
```

### Breakdown:

* `du` → command (disk usage)
* `-sh` → options

  * `-s` = summary
  * `-h` = human-readable format
* `/home/student` → argument (target directory)

---

## 4. Options

* Modify how a command behaves
* Start with:

  * `-` (single dash) → short options
  * `--` (double dash) → long options
* Optional, but often important

---

## 5. Arguments

* The “target” of a command
* Example:

  * file
  * directory
  * user
  * system resource

---

## 6. Terminal (GUI Access)

* Terminal = application that gives access to the shell
* In RHEL GUI:

  * Open terminal from application menu (dock / Red Hat menu)
* Terminal automatically starts **Bash shell**

---

## 7. Tab Completion (VERY IMPORTANT)

* Press **Tab** to auto-complete:

  * commands
  * file paths
  * sometimes options and subcommands

### How it works:

* Press `Tab` once → completes if unique
* Press `Tab` twice → shows all matches

### Example:

```
cd /u + Tab + /sh + Tab
```

→ becomes:

```
cd /usr/share
```

---

## 8. Command Subcommands (Example: podman)

* Some tools have nested commands:

  ```
  podman generate spec
  ```
* Tab helps explore available commands:

  * Press Tab to list possibilities
  * Requires enough uniqueness in typing

---

## 9. Virtual Terminals (TTYs)

Linux has multiple virtual consoles:

| TTY       | Purpose                     |
| --------- | --------------------------- |
| tty1      | Graphical login screen      |
| tty2      | Graphical session (usually) |
| tty3–tty6 | Text-based logins           |

### Switching:

* `Ctrl + Alt + F1–F6`

---

## 10. Remote Access (SSH)

* **SSH (Secure Shell)** allows remote login to Linux systems.
* Command:

  ```
  ssh user@hostname
  ```

### Authentication methods:

* Password
* Public/private key (more secure)

---

## 11. SSH Key Authentication

* Client has:

  * Private key (kept secret)
* Server has:

  * Public key (stored in account)
* Login happens without password if keys match

---

## 12. First-time SSH Login

* SSH shows a **host fingerprint**
* User must confirm:

  ```
  Are you sure you want to continue connecting? (yes/no)
  ```
* Fingerprint is saved for future security checks

### Security warning:

* If fingerprint changes unexpectedly:

  * Could be server reinstall OR
  * Possible attack (man-in-the-middle)

---

## 13. Exiting Sessions

Ways to logout:

* `exit`
* `Ctrl + D` (shortcut)

---

## 14. Switching to Root User

* Command:

  ```
  su -
  ```
* Switches to root (admin user)
* Prompts for root password

### Root prompt:

```
#
```

* Very important: indicates full system access

---

## 15. Useful Keyboard Shortcuts

| Shortcut | Action                 |
| -------- | ---------------------- |
| Ctrl + C | Cancel running command |
| Ctrl + L | Clear screen           |
| Ctrl + D | Logout / exit shell    |

---

## 16. Key Concepts to Remember

* `$` = normal user (safe mode)
* `#` = root (powerful, dangerous if misused)
* Tab completion improves speed and accuracy
* SSH is the standard way to access remote Linux systems
* Virtual terminals allow multiple login sessions
* Always verify SSH fingerprints for security

---
