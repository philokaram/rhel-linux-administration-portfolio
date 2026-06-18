# Investigating and Resolving SELinux Issues — Notes

## 1. The Problem — Breaking a Working Service

### `chcon` — Changing Context Directly

```bash
chcon -R -t default_t /site
```

| Option | Meaning |
|--------|---------|
| `-t` | Set the **type** context |
| `-R` | **Recursive** — apply to all contents |

> ⚠️ `chcon` changes the context **in the inode** but does **NOT** update the SELinux database.

### Result:

```bash
ls -ldZ /site
```

**Output:** `default_t` (not `httpd_sys_content_t`)

### Web server behavior:

- HTTP daemon cannot access files with `default_t`
- Returns the **default page** (not `hello world`)

---

## 2. The Easiest Way — Web Console (Cockpit)

### Why use the web console?

- No complex commands to remember
- Interactive, visual interface
- Provides **ready-made solutions** for SELinux issues

### Steps:

1. Ensure cockpit is running:

```bash
systemctl enable --now cockpit.socket
```

2. Connect to: `https://servera:9090`

3. Log in as `student` (password: `student`)

4. Turn on administrative access (click button)

5. Navigate to **Tools → SELinux**

6. Expand a violation

7. Review the **recommended solutions**

---

## 3. SELinux Violation Solutions in Cockpit

### Solution 1 — Manual context change:

```
semanage fcontext -a -t httpd_sys_content_t "/site(/.*)?"
restorecon -Fvr /site
```

> 💡 This works if you know the correct context.

### Solution 2 — Generate local policy module (2 commands):

Command 1:

```bash
grep -e "denied.*httpd" /var/log/audit/audit.log | audit2allow -m httpd-local > httpd-local.te
```

Command 2:

```bash
audit2allow -R -M httpd-local && semodule -i httpd-local.pp
```

> 💡 This automatically generates a **local policy module** that allows the service to work.

---

## 4. Command-Line Troubleshooting Tools

### `systemctl status setroubleshootd`:

```bash
systemctl status setroubleshootd
```

- Shows recent SELinux denials

### `journalctl -r -u setroubleshootd`:

```bash
journalctl -r -u setroubleshootd
```

| Option | Meaning |
|--------|---------|
| `-r` | Reverse — most recent first |
| `-u` | Unit — filter by systemd unit |

### `journalctl -t setroubleshoot`:

```bash
journalctl -t setroubleshoot
```

| Option | Meaning |
|--------|---------|
| `-t` | Topic — filter by syslog identifier |

### `sealert -l <AVC>`:

```bash
sealert -l "SELinux is preventing /usr/sbin/httpd from getattr access"
```

- Provides detailed analysis and recommended fixes

---

## 5. `chcon` vs `semanage fcontext` — Important Distinction

| Method | What It Does | Persistence |
|--------|--------------|-------------|
| `chcon -t type /path` | Changes context **in the inode only** | ❌ Not guaranteed (can be overwritten by `restorecon`) |
| `semanage fcontext -a -t type "/path(/.*)?"` | Adds rule to **SELinux database** | ✅ Persistent (survives reboots and `restorecon`) |

> 💡 **Best practice:** Use `semanage fcontext` + `restorecon` for permanent context changes.

---

## 6. Working with Audit Logs

### Audit log location:

```bash
/var/log/audit/audit.log
```

### Search for AVC denials (SELinux violations):

```bash
ausearch -m AVC -ts recent
```

| Option | Meaning |
|--------|---------|
| `-m AVC` | Search for AVC (Access Vector Cache) messages |
| `-ts recent` | Time since recent |

### View recent SELinux denials:

```bash
grep "denied.*httpd" /var/log/audit/audit.log
```

---

## 7. Audit2allow — Generate Local Policy

### Step 1 — Generate a policy module:

```bash
grep -e "denied.*httpd" /var/log/audit/audit.log | audit2allow -m httpd-local > httpd-local.te
```

### Step 2 — Build and install the module:

```bash
audit2allow -R -M httpd-local && semodule -i httpd-local.pp
```

> 💡 This creates a **custom policy module** that resolves the SELinux issue permanently.

---

## 8. `restorecon` — The Fix for `chcon`

### If you used `chcon` and want to restore the correct context:

```bash
restorecon -Fvr /site
```

| Option | Meaning |
|--------|---------|
| `-F` | Force — reset even if context is already set |
| `-v` | Verbose — show changes |
| `-r` | Recursive — apply to all contents |

> 💡 `restorecon` applies the correct context from the SELinux database.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Temporary context change (not recommended) | `chcon -t type /path` |
| Add context to database | `semanage fcontext -a -t type "/path(/.*)?"` |
| Apply database context | `restorecon -Fvr /path` |
| View SELinux violations | `journalctl -u setroubleshootd` |
| View AVC denials | `ausearch -m AVC -ts recent` |
| View audit log | `grep denied /var/log/audit/audit.log` |
| Generate policy module | `audit2allow -m module < /path/to/audit.log` |
| Install policy module | `semodule -i module.pp` |
| Web console | `https://server:9090` → Tools → SELinux |

---

# Troubleshooting Workflow

```
1. Service not working
       │
       ▼
2. Check if SELinux is blocking:
   journalctl -u setroubleshootd | tail
       │
       ▼
3. View the AVC denial:
   ausearch -m AVC -ts recent
       │
       ▼
4. Use cockpit OR:
   a) Find correct context and apply:
      semanage fcontext -a -t httpd_sys_content_t "/site(/.*)?"
      restorecon -Fvr /site
   b) Or generate local policy:
      audit2allow -M localpolicy < /var/log/audit/audit.log
      semodule -i localpolicy.pp
       │
       ▼
5. Verify service works ✅
```

---

# Key Takeaways

- **`chcon`** changes context in the inode — **temporary** (not recommended for permanent use).
- **`semanage fcontext`** adds rules to the SELinux database — **persistent** (the right way).
- **`restorecon`** applies the correct context from the database.
- **Web Console (Cockpit)** is the **easiest** way to troubleshoot SELinux:
  - Visual interface
  - Ready-made solutions
  - No need to remember complex commands
- **`setroubleshootd`** logs SELinux denials with recommended solutions.
- **`journalctl -u setroubleshootd`** shows recent SELinux issues.
- **`ausearch -m AVC -ts recent`** searches audit logs for AVC denials.
- **`audit2allow`** generates custom policy modules from audit logs.
- **`semodule -i`** installs custom policy modules.
- **Work smarter, not harder** — use the web console for quick SELinux troubleshooting.
- **Exam tip:** Don't fear SELinux — Cockpit is available on exam systems and can save you time.