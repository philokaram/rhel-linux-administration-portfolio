# Guided Exercise: Create Links Between Files — Notes

## Task 1: Observe the Target File

### Command:

```bash
ls -li files/target.file
```

### What `ls -li` shows:

| Column | Meaning |
|--------|---------|
| `774351` | Inode number (index node) |
| `-rw-rw-r--` | Permissions |
| `1` | **Link count** (number of names for this inode) |
| `student` | Owner |
| `student` | Group |
| `...` | Size, timestamp |
| `files/target.file` | File path |

### Observation:

- Inode = `774351` (yours will be **different** — don't worry!)
- Link count = `1` → one name associated with this inode
- That one name = `files/target.file`

---

## Task 2: Create a Hard Link

### Command:

```bash
ln /home/student/files/target.file /home/student/links/file.hardlink
```

> 💡 Use **tab completion** to avoid typos.

### Breakdown:

| Part | Meaning |
|------|---------|
| `ln` | Create a link (default = hard link) |
| `/home/student/files/target.file` | Source (existing file) |
| `/home/student/links/file.hardlink` | Link name to create |

---

## Task 3: Verify the Hard Link

### Command:

```bash
ls -li files/target.file links/file.hardlink
```

### Expected output:

```
774351 -rw-rw-r-- 2 student student ... files/target.file
774351 -rw-rw-r-- 2 student student ... links/file.hardlink
```

### What this proves:

| Observation | Meaning |
|-------------|---------|
| Same inode (`774351`) | They are the **same file** |
| Link count = `2` | Two names for this inode |
| Same permissions | Metadata is shared |
| Same ownership | Metadata is shared |
| Same timestamp | Metadata is shared |

> 💡 Any content written to `target.file` will appear in `file.hardlink` and vice versa — because they're the **same file**.

---

## Task 4: Create a Symbolic Link (Symlink)

### Goal:

Create `~/tempdir` that points to `/tmp`

### Command:

```bash
ln -s /tmp /home/student/tempdir
```

| Part | Meaning |
|------|---------|
| `ln -s` | Create a **symbolic** (soft) link |
| `/tmp` | Target (what the link points to) |
| `/home/student/tempdir` | Link name (where the symlink lives) |

---

## Task 5: Verify the Symlink

### Command:

```bash
ls -l /home/student/tempdir
```

### Expected output:

```
lrwxrwxrwx 1 student student 4 Aug 26 10:00 /home/student/tempdir -> /tmp
```

### What each part tells us:

| Indicator | Meaning |
|-----------|---------|
| `l` (first character) | It's a **link** (not a regular file) |
| `-> /tmp` | Points to `/tmp` |
| Link count = `1` | Symlinks have their own inode (count not shared) |

---


# Complete Command Reference Table

| Step | Command | Purpose |
|------|---------|---------|
| View inode info | `ls -li files/target.file` | See inode and link count |
| Create hard link | `ln /home/student/files/target.file /home/student/links/file.hardlink` | Add second name for same inode |
| Verify hard link | `ls -li files/target.file links/file.hardlink` | Confirm same inode, link count=2 |
| Create symlink | `ln -s /tmp /home/student/tempdir` | Create symbolic link |
| Verify symlink | `ls -l /home/student/tempdir` | See `l` and `->` |

---

# Expected Observations Checklist

| Task | What to observe | Status |
|------|-----------------|--------|
| 1 | Inode number for `target.file` (e.g., 774351) | ☐ |
| 1 | Link count = 1 | ☐ |
| 3 | Both files have **same** inode | ☐ |
| 3 | Link count increased to **2** | ☐ |
| 3 | Permissions, ownership, timestamps match | ☐ |
| 5 | `ls -l` shows `l` as first character | ☐ |
| 5 | `->` arrow shows target (`/tmp`) | ☐ |

---

# Key Takeaways

- **`ls -li`** shows inode numbers and link counts in one command.
- A **hard link** creates another name for the **same inode** (same file).
- After a hard link, the **link count** increments by 1.
- Hard links = same inode, same permissions, same ownership, same content.
- A **symbolic link** creates a new inode that points to a target **by path**.
- `ls -l` on a symlink shows:
  - `l` (file type = link)
  - `-> target` (where it points)
- Use **hard links** for same-filesystem, space-efficient file sharing.
- Use **symlinks** for cross-filesystem links, directories, or simplifying paths.