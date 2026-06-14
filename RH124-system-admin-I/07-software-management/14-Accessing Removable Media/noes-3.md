# Locating Files on a File System — Notes

## 1. Two Commands for Finding Files

| Command | Speed | Pros | Cons |
|---------|-------|------|------|
| `locate` | Fast | Searches pre-built database | Database can be outdated |
| `find` | Slower | Always up to date, very powerful | More complex syntax |

> 💡 **Ricardo's opinion:** The `locate` command sucks. Use `find` instead.

---

## 2. The `locate` Command — How It Works

### Basic usage:

```bash
locate sshd_config
```

- Returns all matches where the pattern appears anywhere in the path
- Wildcards are **implied** (before and after the pattern)

### Case-insensitive search:

```bash
locate -i SSHD_config
```

### The problem — outdated database:

```bash
touch /root/foo
locate foo           # NOT FOUND — file not in database yet
```

### Fix — update the database:

```bash
updatedb             # Updates the locate database
locate foo           # NOW it finds the file
```

### Why `locate` sucks:

1. **Database must be manually updated** (`updatedb`)
2. New files are **invisible** until you run `updatedb`
3. **Limited** — can only search by name (not by user, size, permissions, etc.)

> 💡 Only use `locate` if you need a **very fast** search and you know the database is up to date.

---

## 3. The `find` Command — The Right Tool

### Why `find` is better:

- Always **up to date** (searches live file system)
- Can search by **many criteria**:
  - Name (case-sensitive or insensitive)
  - Type (file, directory, symlink)
  - User/group ownership
  - Permissions
  - Size
  - Time (access, modification, change)
  - And more...

### Basic syntax:

```bash
find [starting-where] [how] [what] [and-then-what]
```

| Part | Meaning | Example |
|------|---------|---------|
| `starting-where` | Directory to start search | `/`, `/home`, `.` |
| `how` | Search criteria | `-iname`, `-user`, `-size` |
| `what` | The value | `"*.conf"`, `ricardo`, `+1G` |
| `and-then-what` | Action to take (optional) | `-delete`, `-exec` |

> 📌 `find` always searches **recursively** by default.

### Important note about `find` options:

- Most commands use `-` for short options (`-l`, `-a`) and `--` for long options (`--all`)
- **`find` is different** — uses a **single dash** (`-`) even for word options like `-maxdepth`

---

## 4. `find` Examples — By Name

### Case-insensitive search (recommended):

```bash
find / -iname "sshd_config"
```

### Case-sensitive search:

```bash
find / -name "sshd_config"
```

### Using wildcards:

```bash
find / -iname "*sshd_config*"
```

### Limit search depth:

```bash
find / -maxdepth 3 -iname "*.conf"
```

---

## 5. `find` Examples — By Type

| Option | Meaning |
|--------|---------|
| `-type f` | Regular files |
| `-type d` | Directories |
| `-type l` | Symbolic links |

### Find only directories:

```bash
find / -type d -iname "*sshd_config*"
```

### Find only files:

```bash
find / -type f -iname "*sshd_config*"
```

---

## 6. `find` Examples — By User/Owner

### Find by username:

```bash
find / -user mo
```

### Find by UID:

```bash
find / -uid 1002
```

### Suppress permission errors:

```bash
find / -user mo 2>/dev/null
```

> 💡 `2>/dev/null` discards error messages (permission denied).

---

## 7. `find` Examples — By Size

### Size operators:

| Operator | Meaning |
|----------|---------|
| `+` | Greater than |
| `-` | Less than |
| (none) | Exactly |

### Size units:

| Unit | Meaning |
|------|---------|
| `c` | Bytes |
| `k` | Kibibytes (KiB) |
| `M` | Mebibytes (MiB) |
| `G` | Gibibytes (GiB) |

### Examples:

```bash
find / -size +1G          # Larger than 1 GiB
find / -size -500M        # Smaller than 500 MiB
find / -size 150M         # Exactly 150 MiB
```

---

## 8. `find` Examples — By Time

### Access time (`-atime` = days, `-amin` = minutes):

```bash
find / -type f -atime +30      # Last accessed >30 days ago
find / -type f -amin -60       # Last accessed <60 minutes ago
```

### Modification time (`-mtime`/`-mmin`):

```bash
find / -type f -mtime -1       # Modified in last 24 hours
find / -type f -mmin -30       # Modified in last 30 minutes
```

### Change time (`-ctime`/`-cmin`):

```bash
find / -type f -ctime +7       # Metadata changed >7 days ago
```

---

## 9. `find` Examples — By Permissions

### Exact permissions:

```bash
find / -type f -perm 777
```

### Include specific bits (forward slash `/`):

```bash
find / -type f -perm /2000      # SGID bit set
find / -type f -perm /1000      # Sticky bit set
find / -type f -perm /4000      # SUID bit set
```

### Multiple bits:

```bash
find / -type f -perm /6000      # SUID OR SGID set
```

---

## 10. `find` Examples — Actions (`-delete`, `-exec`)

### Delete found files (no confirmation):

```bash
find /var/tmp -name "*.tmp" -delete
```

### Delete with confirmation (`-exec rm -i`):

```bash
find /isos -name "*.iso" -size 150M -exec rm -i {} \;
```

| Part | Meaning |
|------|---------|
| `-exec` | Execute a command |
| `rm -i` | Command to run (interactive) |
| `{}` | Placeholder for each found file |
| `\;` | End of `-exec` command |

### List found files (default action):

```bash
find /home -user ricardo -name "*.iso"
```

---

## 11. Combining Multiple Criteria

### Example: Find files owned by `ricardo`, ending in `.iso`, larger than 1G, and delete them:

```bash
find / -user ricardo -name "*.iso" -type f -size +1G -delete
```

### Example: Find files modified in last hour, owned by `student`:

```bash
find / -type f -mmin -60 -user student
```

---

## 12. The `man find` EXAMPLES Section

```bash
man find
```

Scroll to the **EXAMPLES** section — it's very helpful, especially in exams.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Search by name (case-insensitive) | `find / -iname "pattern"` |
| Search by name (case-sensitive) | `find / -name "pattern"` |
| Search only files | `find / -type f -iname "*.conf"` |
| Search only directories | `find / -type d -iname "*.conf"` |
| Search by user | `find / -user ricardo` |
| Search by UID | `find / -uid 1002` |
| Search by size (>1G) | `find / -size +1G` |
| Search by size (exact 150M) | `find / -size 150M` |
| Search by access time (>30 days) | `find / -atime +30` |
| Search by modification time (<60 min) | `find / -mmin -60` |
| Search by exact permissions | `find / -perm 777` |
| Search by permission bits | `find / -perm /2000` |
| Delete found files | `find / -name "*.tmp" -delete` |
| Execute command on found files | `find / -name "*.iso" -exec ls -l {} \;` |
| Interactive delete | `find / -name "*.iso" -exec rm -i {} \;` |
| Suppress errors | `find / -name "*.conf" 2>/dev/null` |
| Limit search depth | `find / -maxdepth 2 -name "*.conf"` |
| Update locate database | `updatedb` |
| Search with locate | `locate -i pattern` |

---

# `find` Criteria Summary Table

| Search By | Option | Example |
|-----------|--------|---------|
| Name (case-insensitive) | `-iname` | `-iname "foo.txt"` |
| Name (case-sensitive) | `-name` | `-name "foo.txt"` |
| Type | `-type` | `-type f` (file), `-type d` (dir) |
| User | `-user` | `-user ricardo` |
| UID | `-uid` | `-uid 1002` |
| Group | `-group` | `-group wheel` |
| Size | `-size` | `-size +1G`, `-size 150M` |
| Access time (days) | `-atime` | `-atime +30` |
| Access time (minutes) | `-amin` | `-amin -60` |
| Modification time (days) | `-mtime` | `-mtime -1` |
| Modification time (minutes) | `-mmin` | `-mmin +120` |
| Permissions (exact) | `-perm` | `-perm 755` |
| Permissions (any bit) | `-perm /` | `-perm /6000` (SUID or SGID) |
| Depth limit | `-maxdepth` | `-maxdepth 3` |

---

# `locate` vs `find` — Comparison

| Feature | `locate` | `find` |
|---------|----------|--------|
| Database | Pre-built (needs `updatedb`) | None (live search) |
| Up-to-date | No (until `updatedb`) | Yes (always) |
| Search by name | ✅ Yes | ✅ Yes |
| Search by user | ❌ No | ✅ Yes |
| Search by size | ❌ No | ✅ Yes |
| Search by time | ❌ No | ✅ Yes |
| Search by permissions | ❌ No | ✅ Yes |
| Execute actions | ❌ No | ✅ Yes (`-exec`, `-delete`) |
| Speed | Very fast | Slower (but worth it) |

---

# Key Takeaways

- **`locate`** is fast but **limited** — database must be updated with `updatedb`. New files are invisible until you update.
- **`find`** is **powerful and always up to date** — learn it well.
- **`find` syntax:** `find [starting-where] [how] [what] [and-then-what]`
- Always start with a broad search, then narrow down with additional criteria.
- Use `-iname` instead of `-name` unless you need case-sensitivity (it's part of muscle memory).
- Use `2>/dev/null` to suppress "permission denied" errors.
- The **EXAMPLES** section in `man find` is invaluable — especially in exams.
- **`find` can delete files** — be careful with `-delete` and `-exec rm`.
- The `locate` command is covered because it's in the training, but **`find` is what you'll actually use.**