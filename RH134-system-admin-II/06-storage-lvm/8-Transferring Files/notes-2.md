# Synchronizing Content Between Systems — Notes

## 1. What Is Rsync?

- **rsync** = **R**emote **sync**hronization
- Designed for efficiently keeping files/directories synchronized
- Uses a **delta-transfer algorithm** — only transfers differences, not entire files

### Why rsync is preferred:

| Feature | Benefit |
|---------|---------|
| Delta-transfer | Only sends changed parts of files |
| Resume support | Can resume interrupted transfers |
| Versatile | Many options for permissions, ownership, timestamps |
| Secure | Uses SSH for encrypted transfers |
| Fast | Much faster for subsequent transfers |

> 💡 **Analogy:** If you update one paragraph in a 100-page document, rsync only sends that paragraph, not all 100 pages.

---

## 2. Rsync vs SCP vs SFTP

| Feature | Rsync | SCP | SFTP |
|---------|-------|-----|------|
| Delta-transfer | ✅ Yes | ❌ No | ❌ No |
| Resume support | ✅ Yes | ❌ No | ✅ Yes (`re-get`) |
| Interactive | ❌ No | ❌ No | ✅ Yes |
| Preserves permissions | ✅ Yes | ✅ Yes | ❌ No |
| Uses SSH | ✅ Yes | ✅ Yes | ✅ Yes |

> 💡 **Rsync is the ultimate tool** for file synchronization.

---

## 3. Basic Rsync Syntax

```bash
rsync -avz source/ destination/
```

| Option | Meaning |
|--------|---------|
| `-a` | **Archive mode** (preserves permissions, ownership, timestamps, symlinks) |
| `-v` | **Verbose** — show what's being transferred |
| `-z` | **Compress** data during transfer (saves bandwidth) |
| `-P` | **Progress** + keep partial files (resume support) |
| `--delete` | Remove files on destination that don't exist on source |

### Archive mode breakdown (`-a` = `-rlptgoD`):

| Option | Meaning |
|--------|---------|
| `-r` | Recursive |
| `-l` | Copy symlinks as symlinks |
| `-p` | Preserve permissions |
| `-t` | Preserve timestamps |
| `-g` | Preserve group |
| `-o` | Preserve owner |
| `-D` | Preserve device files (special files) |

---

## 4. Rsync Examples

### Copy a file from local to remote:

```bash
rsync -avP bigfile serverb:
```

| Part | Meaning |
|------|---------|
| `bigfile` | Source file |
| `serverb:` | Remote host (colon after hostname) |
| (nothing after `:`) | Copy to remote user's home directory |

### Copy to a specific remote directory:

```bash
rsync -avP bigfile serverb:/var/tmp/
```

### Copy with a different username:

```bash
rsync -avP bigfile devops@serverb:
```

### Copy a directory (without trailing slash):

```bash
rsync -av /etc shadow/
```

**Result:** `shadow/etc/` (entire `/etc` directory inside `shadow`)

### Copy a directory (with trailing slash):

```bash
rsync -av /etc/ shadow/
```

**Result:** Contents of `/etc` copied directly into `shadow/` (no `etc/` subdirectory)

---

## 5. The Trailing Slash Difference — Critical!

| Command | Result |
|---------|--------|
| `rsync -av /etc shadow/` | Creates `shadow/etc/` with contents |
| `rsync -av /etc/ shadow/` | Copies **contents** of `/etc/` into `shadow/` |

> 💡 **Important:** Trailing slash (`/`) on the source means "copy the **contents**" of the directory, not the directory itself.

---

## 6. Rsync — Delta-Transfer in Action

### Step 1 — Create a 100 MiB file:

```bash
dd if=/dev/zero of=bigfile bs=1M count=100
```

### Step 2 — Copy to remote:

```bash
rsync -avP bigfile serverb:
```

**Literal data transferred:** 100 MiB (entire file)

### Step 3 — Add 16 MiB of data to the file:

```bash
cat /usr/share/dict/words >> bigfile
```

### Step 4 — Rsync again:

```bash
rsync -avP bigfile serverb:
```

**Literal data transferred:** ~16 MiB (ONLY the changes!)

> 💡 Rsync only sends the **differences** — not the entire file.

---

## 7. Rsync Options — Quick Reference

| Option | Meaning |
|--------|---------|
| `-a` | Archive mode (preserve all) |
| `-v` | Verbose |
| `-z` | Compress during transfer |
| `-P` | Show progress + partial file support |
| `--delete` | Remove extra files on destination |
| `--dry-run` | Simulate, don't actually transfer |
| `--exclude` | Exclude files matching pattern |
| `-e ssh` | Use SSH as transport (default) |

---

## 8. Rsync — Backup Use Case

### Mirror directories (delete files on destination that aren't on source):

```bash
rsync -av --delete /source/ /destination/
```

> ⚠️ **Use with caution:** `--delete` removes files on the destination.

### Dry run (simulate first):

```bash
rsync -av --delete --dry-run /source/ /destination/
```

### Exclude certain files:

```bash
rsync -av --exclude="*.tmp" /source/ /destination/
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Copy file to remote (home dir) | `rsync -avP file serverb:` |
| Copy file to remote (specific dir) | `rsync -avP file serverb:/var/tmp/` |
| Copy directory (preserve structure) | `rsync -av /etc shadow/` |
| Copy directory contents (no subdir) | `rsync -av /etc/ shadow/` |
| Copy with compression | `rsync -avzP file serverb:` |
| Copy and delete extras | `rsync -av --delete /src/ /dest/` |
| Simulate (dry run) | `rsync -av --dry-run /src/ /dest/` |
| Exclude patterns | `rsync -av --exclude="*.tmp" /src/ /dest/` |
| View rsync man page | `man rsync` |

---

# Trailing Slash — Quick Reference

```
rsync -av /source/ dest/
    │       │
    │       └── Source has trailing slash → copy CONTENTS only
    │
    └── Destination directory (will receive contents)

rsync -av /source dest/
    │       │
    │       └── Source has NO trailing slash → copy SOURCE DIRECTORY
    │
    └── Destination directory (source dir will be created inside)
```

### Example:

```bash
rsync -av /etc/ shadow/   # shadow/ contains hosts, fstab, etc.
rsync -av /etc shadow/    # shadow/ contains etc/ with contents
```

---

# Rsync vs SCP — Real-World Comparison

| Scenario | SCP | Rsync |
|----------|-----|-------|
| First transfer (100 MiB) | 100 MiB | 100 MiB |
| Add 16 MiB to file | **100 MiB** (entire file) | **16 MiB** (only changes) |
| Add another 10 MiB | **100 MiB** (entire file) | **10 MiB** (only changes) |
| Connection drops | Start over | Resume from where it left |

---

# Key Takeaways

- **Rsync** is the preferred tool for file synchronization.
- **Delta-transfer algorithm** = only sends the differences (huge time/bandwidth savings).
- **Archive mode (`-a`)** preserves permissions, ownership, timestamps, symlinks.
- **Trailing slash (`/`)** on source:
  - **With slash:** copy **contents** of directory.
  - **Without slash:** copy **directory itself**.
- **`-P`** = progress + partial file support (resume).
- **`-z`** = compress during transfer (saves bandwidth).
- **`--delete`** = remove files on destination that don't exist on source (use with caution).
- **`--dry-run`** = simulate transfer without actually copying (safety check).
- **Rsync uses SSH** — secure, supports key-based authentication.
- **Personal recommendation:** Use `rsync` instead of `scp` or `sftp` for most transfer tasks.