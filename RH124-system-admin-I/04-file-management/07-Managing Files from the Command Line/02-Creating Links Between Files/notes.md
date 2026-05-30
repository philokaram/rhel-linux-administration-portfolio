# Creating Links Between Files — Notes

## 1. How Computing Identifies Components

| Component | Identifier |
|-----------|------------|
| Processes | Process ID (PID) |
| Users | User ID (UID) |
| Groups | Group ID (GID) |
| **Files** | **Inode** (Index Node) |

> ❌ Not "File ID" — it's called an **inode**.

---

## 2. What Is an Inode?

- Generated for **each file**
- Size: **256 bytes** of metadata (on XFS file system — varies by filesystem)

### What's stored in an inode:

| Metadata | Example |
|----------|---------|
| File type | Regular file, directory, block file, character device |
| Permissions | `rwxr-xr-x` |
| Ownership | UID, GID |
| Timestamps | Access, modify, change |
| **Link count** | Number of names pointing to this inode |
| **Pointers to data blocks** | Where the actual file data lives |

### What's NOT stored in an inode:

- The file name
- The actual file data (just pointers to it)

> 💡 A 1 GiB file's data cannot fit in 256 bytes — hence the **pointers**.

---

## 3. File Allocation Table

A **mapping** of inodes to file paths:

| Inode | File Path |
|-------|-----------|
| 1001731 | `/foo/bar` |
| 1044217 | `/baz/bongle` |

### Multiple names for the same inode:

| Inode | File Path |
|-------|-----------|
| 1001731 | `/foo/bar` |
| 1001731 | `/foo/rab` |

> Both paths point to the **same file** — same metadata, same data.

---

## 4. Hard Links

### What is a hard link?

Another name for the **same inode**.

### Creating a hard link:

```bash
ln source_file link_name
```

### Example:

```bash
touch foo
echo "hello world" > foo
ln foo bar
```

### View inode information:

```bash
ls -li
```

**Output:**
```
8337248 -rw-rw-r-- 2 student student 12 Aug 26 10:00 bar
8337248 -rw-rw-r-- 2 student student 12 Aug 26 10:00 foo
```

| Field | Meaning |
|-------|---------|
| `8337248` | Inode number (same for both!) |
| `2` | Link count |
| `12` | File size |

### Link count explained:

| Scenario | Link Count |
|----------|------------|
| One name for the inode | 1 |
| Two names for the inode | 2 |
| Three names for the inode | 3 |

### Verify same content:

```bash
cat foo   # hello world
cat bar   # hello world
```

### Hard link limitations:

| Limitation | Why? |
|------------|------|
| Same file system only | Inodes are unique **per file system** (inode 8337248 on another FS is a different file) |
| Cannot link across directories | (Technically can, but with restrictions) |
| Cannot link to directories | (Prevents circular references) |

> 💡 Hard links = same inode, same data, same metadata, just **multiple names**.

---

## 5. Symbolic Links (Symlinks)

### What is a symlink?

A **special file** that points to another file or directory by **path**.

### Creating a symlink:

```bash
ln -s target_path link_name
```

| Option | Meaning |
|--------|---------|
| `-s` | Symbolic (soft) link |

### Use case 1: Simplify deep directory paths

**Problem:** Complex directory for user websites:
```
/site/prod/users/public/student/html
```

**Solution:** Create a symlink in the user's home directory:

```bash
ln -s /site/prod/users/public/student/html ~student/html
```

**Result:** User just goes to `~/html` instead of typing the full path.

### Use case 2: Link across different file systems

**Problem:** Log files are in `/var/log/httpd/` (maybe different filesystem than `/home/`)

**Solution:** Symlinks work across filesystems:

```bash
ln -s /var/log/httpd/access_log ~student/access_log
ln -s /var/log/httpd/error_log ~student/error_log
```

### Visual indicators of symlinks:

| Clue | Meaning |
|------|---------|
| Color (light blue/cyan) | It's a link |
| `ls -l` output | Shows `->` pointing to target |

```bash
ls -l ~student/html
```

**Output:**
```
lrwxrwxrwx 1 root root 42 Aug 26 10:00 html -> /site/prod/users/public/student/html
```

Note the `l` at the beginning = **l**ink.

---

## 6. Hard Link vs Symbolic Link — Comparison

| Feature | Hard Link | Symbolic Link (Symlink) |
|---------|-----------|------------------------|
| Command | `ln target link` | `ln -s target link` |
| Creates new inode? | No (same inode) | Yes (new inode) |
| Works across file systems? | No | Yes |
| Can link to directories? | No | Yes |
| If target is deleted... | Link still works (data remains) | Link is broken (dangling) |
| Link count increments? | Yes | No |
| Visual indicator | Normal file (no special color) | Special color + `->` arrow |

---

## 7. Practical Examples

### Example 1: Creating a hard link

```bash
touch file1
ln file1 file2
ls -li file1 file2
# Same inode, link count = 2
```

### Example 2: Creating a symlink to a directory

```bash
ln -s /var/log ~/logs
cd ~/logs
pwd   # /home/student/logs (but actually shows /var/log)
```

### Example 3: Creating a symlink to a file across filesystems

```bash
ln -s /var/log/httpd/access_log ~/access_log
```

### Example 4: Broken symlink (target deleted)

```bash
ln -s /tmp/missing_file ~/broken
rm /tmp/missing_file
cat ~/broken   # No such file or directory
```

---

## 8. Viewing Inode Information

| Command | What it shows |
|---------|---------------|
| `ls -i` | Inode numbers only |
| `ls -li` | Inode numbers + long listing |
| `stat filename` | Detailed inode information |

### Example `stat` output:

```bash
stat foo
```

```
  File: foo
  Size: 12              Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 8337248     Links: 2
Access: (0664/-rw-rw-r--)  Uid: (1000/student)   Gid: (1000/student)
Access: 2025-08-26 10:00:00
Modify: 2025-08-26 10:00:00
Change: 2025-08-26 10:00:00
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Create a hard link | `ln source link` |
| Create a symlink | `ln -s target link` |
| View inode numbers | `ls -i` |
| View inodes + details | `ls -li` |
| View detailed inode info | `stat filename` |
| Delete a link (unlink) | `rm linkname` or `unlink linkname` |

---

# Inode & Link Count Summary

| Action | Effect on Link Count |
|--------|---------------------|
| Create new file | Link count = 1 |
| Create hard link | Link count increases by 1 |
| Remove a hard link (rm) | Link count decreases by 1 |
| Create symlink | Link count unchanged (symlink has its own inode) |
| Delete target of symlink | Symlink becomes broken (dangling) |

---

# Key Takeaways

- **Inodes** = metadata containers for files (256 bytes on XFS).
- **Inodes store**: permissions, ownership, timestamps, link count, pointers to data blocks.
- **Inodes do NOT store**: file name or file data.
- **Hard links** = multiple names for the **same inode** (same file system only).
- **Symbolic links** = special files that point to a **path** (works across file systems).
- **`ls -li`** shows inode numbers and link counts.
- **Link count** = number of hard links pointing to an inode.
- Symlinks are visually distinct (color + `->` arrow in `ls -l`).
- Use **hard links** for same-filesystem, space-efficient duplicates.
- Use **symlinks** for cross-filesystem links, directories, or simplifying paths.