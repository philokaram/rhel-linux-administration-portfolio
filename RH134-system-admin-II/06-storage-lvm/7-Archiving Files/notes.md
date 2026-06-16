# Managing Compressed TAR Archives — Notes

## 1. What Is TAR?

- **TAR** = **T**ape **AR**chiver (originally for tape backups)
- Collapses a directory structure into a **single file**
- TAR does **not** compress by default — it only archives

> 💡 Think of TAR as a way to bundle many files into one file (like a ZIP without compression).

---

## 2. Compression Tools — Comparison

| Tool | Option | Compression Ratio | CPU Usage | File Extension |
|------|--------|-------------------|-----------|----------------|
| **gzip** | `-z` | Worst | Lowest | `.tar.gz` or `.tgz` |
| **bzip2** | `-j` | Medium | Medium | `.tar.bz2` or `.tbz2` |
| **xz** | `-J` | **Best** | **Highest** | `.tar.xz` or `.txz` |

### Recommendation:

- **Fast compression → gzip**
- **Balance → bzip2**
- **Maximum compression → xz** (but uses more CPU)

> ⚠️ On CPU-constrained systems, avoid `xz` — it adds significant load.

---

## 3. Creating a TAR Archive (No Compression)

```bash
tar -cf etc.tar /etc
```

| Option | Meaning |
|--------|---------|
| `-c` | **C**reate an archive |
| `-f` | **F**ile (specify the archive file) |

**Note:** The leading forward slash (`/`) is **removed** from member names to prevent overwriting system files when extracting.

---

## 4. Creating Compressed Archives

### gzip compression (`.tar.gz`):

```bash
tar -czf etc.tar.gz /etc
```

### bzip2 compression (`.tar.bz2`):

```bash
tar -cjf etc.tar.bz2 /etc
```

### xz compression (`.tar.xz`):

```bash
tar -cJf etc.tar.xz /etc
```

---

## 5. Viewing Archive Contents — `tar -t`

### List contents:

```bash
tar -tf etc.tar.gz
```

### View with pager:

```bash
tar -tf etc.tar.gz | less
```

| Option | Meaning |
|--------|---------|
| `-t` | **T**est/List contents |

> 💡 The leading forward slash (`/`) is removed from paths — files are stored as `etc/hosts`, not `/etc/hosts`.

---

## 6. Extracting Archives — `tar -x`

### Automatic compression detection:

> 💡 `tar` can **automatically detect** the compression type — no need to specify `-z`, `-j`, or `-J` when extracting.

### Extract to current directory:

```bash
tar -xf etc.tar.gz
```

### Extract to a specific directory:

```bash
tar -xf etc.tar.xz -C /var/tmp
```

| Option | Meaning |
|--------|---------|
| `-x` | E**x**tract |
| `-C` | **C**hange directory (where to extract) |

---

## 7. Extracting a Single File

### Step 1 — Check if the file exists in the archive:

```bash
tar -tf etc.tar.gz | grep hosts
```

**Output:** `etc/hosts`

### Step 2 — Extract only that file:

```bash
tar -xf etc.tar.gz etc/hosts
```

### Result:

- Only `etc/hosts` is extracted
- Other files remain in the archive

---

## 8. TAR Command — Quick Reference

| Operation | Command |
|-----------|---------|
| Create (no compression) | `tar -cf archive.tar /path` |
| Create with gzip | `tar -czf archive.tar.gz /path` |
| Create with bzip2 | `tar -cjf archive.tar.bz2 /path` |
| Create with xz | `tar -cJf archive.tar.xz /path` |
| List contents | `tar -tf archive.tar.gz` |
| Extract (autodetect) | `tar -xf archive.tar.gz` |
| Extract to directory | `tar -xf archive.tar.gz -C /target` |
| Extract single file | `tar -xf archive.tar.gz etc/hosts` |

---

## 9. File Size Comparison Example

| Archive | Size | Compression |
|---------|------|-------------|
| Original `/etc` directory | 25 MB | (uncompressed) |
| `etc.tar` | 23 MB | TAR only (no compression) |
| `etc.tar.gz` | 5.6 MB | gzip |
| `etc.tar.bz2` | 4.8 MB | bzip2 |
| `etc.tar.xz` | 4.1 MB | xz (best compression) |

> 💡 `xz` offers the **smallest file size** but uses the **most CPU**.

---

## 10. Key TAR Options — Memory Aid

| Option | Meaning | Mnemonic |
|--------|---------|----------|
| `-c` | Create | **C**reate |
| `-x` | Extract | E**x**tract |
| `-t` | List/Test | **T**est |
| `-f` | File | **F**ile (archive name) |
| `-C` | Change directory | **C**hange |
| `-z` | gzip compression | (not intuitive — check man page) |
| `-j` | bzip2 compression | (not intuitive — check man page) |
| `-J` | xz compression | (not intuitive — check man page) |

> 💡 **Pro tip:** The compression options (`-z`, `-j`, `-J`) aren't intuitive. Use `man tar` if you forget.

---

## 11. Man Page — Your Friend

```bash
man tar
```

- Section 1 (user commands)
- Contains all options and examples
- Search for `/EXAMPLE` to find practical examples

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Create TAR archive | `tar -cf archive.tar /path` |
| Create TAR with gzip | `tar -czf archive.tar.gz /path` |
| Create TAR with bzip2 | `tar -cjf archive.tar.bz2 /path` |
| Create TAR with xz | `tar -cJf archive.tar.xz /path` |
| List archive contents | `tar -tf archive.tar` |
| List with pager | `tar -tf archive.tar \| less` |
| Extract archive | `tar -xf archive.tar` |
| Extract to directory | `tar -xf archive.tar -C /target` |
| Extract single file | `tar -xf archive.tar etc/hosts` |
| View tar man page | `man tar` |

---

# Compression Tool Comparison

| Tool | Option | Extension | Speed | Compression Ratio |
|------|--------|-----------|-------|-------------------|
| gzip | `-z` | `.tar.gz`, `.tgz` | Fast | Low |
| bzip2 | `-j` | `.tar.bz2`, `.tbz2` | Medium | Medium |
| xz | `-J` | `.tar.xz`, `.txz` | Slow | High |

---

# Key Takeaways

- **TAR** = Tape ARchiver — bundles files into one file, **no compression by default**.
- **Compression tools:** `gzip` (fast, low compression), `bzip2` (balanced), `xz` (slow, best compression).
- **Create** with: `tar -c[f][z/j/J] archive.tar.[gz/bz2/xz] /path`
- **List** with: `tar -tf archive.tar.[gz/bz2/xz]`
- **Extract** with: `tar -xf archive.tar.[gz/bz2/xz]` (automatic compression detection).
- **Extract to directory:** `tar -xf archive.tar.gz -C /target`.
- **Extract single file:** `tar -xf archive.tar.gz etc/hosts`.
- Leading forward slash (`/`) is removed from paths to prevent overwriting system files.
- Use `man tar` for all options — you don't need to memorize everything.
- **CPU impact:** `xz` uses the most CPU — avoid on CPU-constrained systems.