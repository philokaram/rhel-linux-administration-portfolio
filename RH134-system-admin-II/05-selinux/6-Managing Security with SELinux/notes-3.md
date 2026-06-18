# Tuning SELinux Policy with Booleans — Notes

## 1. What Are SELinux Booleans?

- **Booleans** = predefined SELinux security settings
- Can be turned **on** or **off** (like a light switch)
- Allow administrators to modify SELinux policy behavior **without** writing new rules

### Categories in SELinux policy:

| Category | Managed By |
|----------|------------|
| Files | `semanage fcontext` |
| Processes/Domains | `semanage permissive` |
| Users | `semanage user` |
| **Booleans** | `setsebool`, `getsebool` |
| Ports | `semanage port` |

> 💡 Booleans are the simplest way to tune SELinux behavior.

---

## 2. Viewing Booleans — `getsebool`

### View all booleans:

```bash
getsebool -a
```

### View specific boolean:

```bash
getsebool httpd_use_nfs
```

**Output:**
```
httpd_use_nfs --> off
```

### Pipe to `less` for browsing:

```bash
getsebool -a | less
```

### Search for a specific category:

```bash
getsebool -a | grep httpd
```

> 💡 `getsebool` gives a **snapshot** of the current state.

---

## 3. Viewing Booleans with Descriptions — `semanage boolean -l`

### List all booleans with descriptions:

```bash
semanage boolean -l
```

**Output format:**
```
SELinux boolean                State  Default Description
httpd_use_nfs                  (off , off)  Allow httpd to use nfs
```

### Search for a specific boolean:

```bash
semanage boolean -l | grep httpd_use_nfs
```

### List customizations only:

```bash
semanage boolean -l -C
```

> 💡 `semanage boolean -l` shows **current state**, **default state**, and a **description**.

---

## 4. Setting Booleans — `setsebool`

### Temporary change (lost on reboot):

```bash
setsebool httpd_use_nfs on
```

### Persistent change (survives reboots):

```bash
setsebool -P httpd_use_nfs on
```

| Option | Meaning |
|--------|---------|
| `-P` | **Persistent** — writes to the SELinux policy database |

> ⚠️ Always use `-P` unless you're just testing temporarily.

### Turn off a boolean:

```bash
setsebool -P httpd_use_nfs off
```

---

## 5. Common SELinux Booleans for HTTP

| Boolean | Purpose |
|---------|---------|
| `httpd_use_nfs` | Allow httpd to serve content from NFS shares |
| `httpd_use_cifs` | Allow httpd to serve content from CIFS/SMB shares |
| `httpd_enable_homedirs` | Allow httpd to serve content from user home directories |
| `httpd_can_network_connect` | Allow httpd to make network connections |
| `httpd_can_network_connect_db` | Allow httpd to connect to databases |
| `httpd_tty_comm` | Allow httpd to communicate with terminals |

---

## 6. Viewing Customizations — `semanage boolean -l -C`

### List only booleans that have been customized:

```bash
semanage boolean -l -C
```

**Output example:**
```
httpd_use_nfs               (on, off)  Allow httpd to use nfs
```

| Field | Meaning |
|-------|---------|
| First value in `( )` | **Current** state |
| Second value in `( )` | **Default** state (from policy) |

> 💡 This shows which booleans have been changed from their defaults.

---

## 7. Use Case — Web Server with NFS Content

### Scenario:

- Web server configured to serve content from an NFS-mounted directory
- Content is accessible, but web server returns 403 Forbidden
- SELinux is blocking access

### Symptom:

```bash
getsebool httpd_use_nfs
```

**Output:** `httpd_use_nfs --> off`

### Solution:

```bash
setsebool -P httpd_use_nfs on
```

### Verify:

```bash
getsebool httpd_use_nfs
```

**Output:** `httpd_use_nfs --> on`

### Result:

- Web server can now serve content from NFS
- No other changes needed

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View all booleans | `getsebool -a` |
| View specific boolean | `getsebool httpd_use_nfs` |
| View booleans with descriptions | `semanage boolean -l` |
| View booleans with descriptions (search) | `semanage boolean -l \| grep httpd` |
| View customizations only | `semanage boolean -l -C` |
| Set boolean (temporary) | `setsebool httpd_use_nfs on` |
| Set boolean (persistent) | `setsebool -P httpd_use_nfs on` |
| Turn boolean off | `setsebool -P httpd_use_nfs off` |

---

# Boolean Workflow

```
1. Identify the problem
   Service not working as expected

2. Check relevant boolean
   getsebool httpd_use_nfs → off

3. Understand what the boolean does
   semanage boolean -l | grep httpd_use_nfs
   → "Allow httpd to use nfs"

4. Enable the boolean persistently
   setsebool -P httpd_use_nfs on

5. Verify
   getsebool httpd_use_nfs → on

6. Test service
   Service now works ✅
```

---

# Key Takeaways

- **Booleans** = predefined SELinux settings that can be toggled on/off.
- **`getsebool`** = view current boolean state.
- **`semanage boolean -l`** = view booleans with descriptions and default states.
- **`setsebool`** = change boolean state.
- **Always use `-P`** for persistent changes (survives reboots).
- **Use `-C`** with `semanage boolean -l` to see only customizations.
- Booleans are the **easiest** way to tune SELinux — no complex policy writing needed.
- Common use cases: NFS/CIFS access, network connections, user home directories.
- If a service is blocked by SELinux, check relevant booleans before writing custom rules.