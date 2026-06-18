# Controlling SELinux File Contexts — Notes

## 1. The Rules Database

SELinux policy rules come from files below:

```bash
/etc/selinux/targeted/context/files/
```

### View the file contexts database:

```bash
cat /etc/selinux/targeted/context/files/file_contexts | less
```

**Format:** `path pattern → SELinux label (context)`

> 💡 This database defines the **expected** SELinux context for files and directories.

---

## 2. SELinux Modes Recap

| Mode | Rules Database Loaded? | Rules Enforced? | Logging? |
|------|----------------------|-----------------|----------|
| **Enforcing** | ✅ Yes | ✅ Yes | ✅ Yes |
| **Permissive** | ✅ Yes | ❌ No (complain only) | ✅ Yes |
| **Disabled** | ❌ No | ❌ No | ❌ No |

> 💡 **Permissive mode** = "complain mode" — rules are loaded but not enforced (good for troubleshooting).

---

## 3. How SELinux Contexts Are Assigned

### When a new file is created:

- It **inherits** the context from its **parent directory**
- Unless there's a **specific rule** in the policy database

### Example:

| File | Location | Inherited Context | Directory Context |
|------|----------|-------------------|-------------------|
| `foo` | `/root/` | `admin_home_t` | `admin_home_t` |
| `bar` | `/usr/share/doc/` | `usr_t` | `usr_t` |
| `baz` | `/etc/` | `etc_t` | `etc_t` |

---

## 4. Moving vs Copying — Context Behavior

| Operation | Context Behavior |
|-----------|------------------|
| **Move (`mv`)** | **Retains** the original context |
| **Copy (`cp`)** | **Inherits** context of the destination directory (by default) |
| **Copy with `--preserve=context`** | **Retains** the original context |

### Example:

```bash
# Move — retains context
mv /root/foo /var/tmp/

# Copy — inherits destination context (default)
cp /usr/share/doc/bar /var/tmp/

# Copy — preserves original context
cp --preserve=context /etc/baz /var/tmp/
```

---

## 5. Viewing SELinux Contexts

### File/directory context:

```bash
ls -Z /path/to/file
```

### Directory context (not its contents):

```bash
ls -ldZ /site
```

### Recursive view:

```bash
ls -lZ /var/www/
```

---

## 6. The `default_t` Context

- When a file/directory has **no specific** SELinux context rule
- It gets the **`default_t`** type
- Services may not be able to access `default_t` files

> ⚠️ If a service can't access files with `default_t`, you need to set the correct context.

---

## 7. Setting a Specific Domain to Permissive Mode

### Problem:

- Entire system in `Enforcing` mode
- A specific service (like `httpd`) is blocked by SELinux
- You don't want to put the entire system in Permissive mode

### Solution — Put only the service domain in Permissive:

```bash
# Add httpd_t domain to permissive mode
semanage permissive -a httpd_t

# Verify
semanage permissive -l

# Remove from permissive mode when done
semanage permissive -d httpd_t
```

> 💡 This is **persistent** — survives reboots until removed.

---

## 8. Adding a File Context — `semanage fcontext -a`

### Step 1 — Add the context to the SELinux database:

```bash
semanage fcontext -a -t httpd_sys_content_t "/site(/.*)?"
```

| Part | Meaning |
|------|---------|
| `-a` | Add |
| `-t` | Type context |
| `httpd_sys_content_t` | The context type to assign |
| `"/site(/.*)?"` | Regex: `/site` and all its contents |

### The regex explained:

| Pattern | Meaning |
|---------|---------|
| `/site` | Exact match for `/site` |
| `(/.*)` | Slash + any characters |
| `?` | Make the previous group **optional** |

> 💡 The `?` makes the rule apply to both the directory itself AND its contents.

---

## 9. Applying the Context — `restorecon`

### `semanage fcontext` only updates the **database** — it does NOT change existing files.

### Apply the context to files:

```bash
restorecon -Fvr /site
```

| Option | Meaning |
|--------|---------|
| `-F` | Force — reset even if context is already set |
| `-v` | Verbose — show changes |
| `-r` | Recursive — apply to all contents |

### Verify:

```bash
ls -ldZ /site
ls -lZ /site/index.html
```

**Expected:** `httpd_sys_content_t`

---

## 10. Common Web Server Contexts

| Context | Purpose |
|---------|---------|
| `httpd_sys_content_t` | Static web content (HTML, images) |
| `httpd_sys_script_exec_t` | CGI scripts |
| `httpd_sys_rw_content_t` | Writable content (uploads) |
| `httpd_log_t` | HTTP log files |

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View file context | `ls -Z /path` |
| View directory context | `ls -ldZ /path` |
| View SELinux database | `ls -Z` (or `matchpathcon`) |
| Add file context to database | `semanage fcontext -a -t type "/pattern(/.*)?"` |
| List file contexts in database | `semanage fcontext -l | grep /site` |
| Apply contexts from database | `restorecon -Fvr /path` |
| Set domain to permissive | `semanage permissive -a httpd_t` |
| List permissive domains | `semanage permissive -l` |
| Remove from permissive | `semanage permissive -d httpd_t` |
| View HTTP daemon context | `ps -Z | grep httpd` |

---

# File Context Workflow

```
1. Service can't access files
              │
              ▼
2. Check context: ls -Z /site
   → Found: default_t (not accessible by httpd)
              │
              ▼
3. Find correct context: ls -Z /var/www/html
   → Found: httpd_sys_content_t
              │
              ▼
4. Add rule to database:
   semanage fcontext -a -t httpd_sys_content_t "/site(/.*)?"
              │
              ▼
5. Apply rule to files:
   restorecon -Fvr /site
              │
              ▼
6. Verify: ls -lZ /site
   → Now: httpd_sys_content_t
              │
              ▼
7. Service works! ✅
```

---

# Key Takeaways

- **SELinux context** = `user:role:type:sensitivity:category` (focus on `_t` type).
- **New files inherit** context from their parent directory.
- **Move (`mv`)** preserves the original context.
- **Copy (`cp`)** inherits the destination directory's context (default).
- **Copy with `--preserve=context`** preserves the original context.
- **`default_t`** = no specific context rule → services may not access it.
- **Permissive mode per domain** = `semanage permissive -a domain_t` (don't set entire system to permissive).
- **`semanage fcontext -a`** adds rules to the SELinux database.
- **`restorecon -Fvr`** applies database rules to files.
- **Regex `/site(/.*)?`** applies rule to directory + all contents.
- **Common web context:** `httpd_sys_content_t`.