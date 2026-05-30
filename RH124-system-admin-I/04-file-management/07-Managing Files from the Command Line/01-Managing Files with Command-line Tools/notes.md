# Managing Files with Command Line Tools — Notes

## 1. Core Philosophy

> **"Work smarter, not harder."**

Use tools and techniques that:
- Enable your success
- Reduce time to market
- Improve productivity
- Reduce human error

---

## 2. Creating Directories (`mkdir`)

### Basic usage:

```bash
mkdir -p site/rgdacosta/html
```

| Option | Meaning |
|--------|---------|
| `-p` | Create parent directories if needed |
| `-v` | Verbose (show what was created) |

---

## 3. Argument History Trick — `Esc + .`

### Problem:
You just created a directory and want to `cd` into it.

### Solution:

```bash
mkdir -p site/rgdacosta/html
cd Esc + .        # Inserts the last argument from previous command
```

**Result:** `cd site/rgdacosta/html`

> 💡 `Esc` then `.` (or `Alt + .`) = insert last argument of previous command.

---

## 4. Creating Multiple Files — Brace Expansion

### Instead of:

```bash
touch file1.htm
touch file2.htm
# ... 98 more times ...
```

### Use brace expansion `{start..end}`:

```bash
touch file{1..100}.htm
```

**Creates:** `file1.htm`, `file2.htm`, ... `file100.htm`

### Brace expansion syntax:

```bash
{start..end}        # Numbers: {1..100}
{start..end..step}  # With step: {1..100..2}
{a..z}              # Letters: {a..z}
```

---

## 5. Renaming Files — The `rename` Command

### Problem: You have 100 `.htm` files but need `.html`

### Bad solution (don't do this):

```bash
mv file1.htm file1.html   # One down, 99 to go 😭
```

### Good solution — `rename`:

```bash
rename .htm .html *
```

### rename syntax:

```bash
rename <what_to_find> <replace_with> <files>
```

| Part | Example | Meaning |
|------|---------|---------|
| What to find | `.htm` | Pattern to match |
| Replace with | `.html` | Replacement text |
| Files | `*` | All files in current directory |

### Example with specific pattern:

```bash
rename .htm .html file*.htm   # Only rename files starting with "file"
```

---

## 6. Moving Files — The `mv` Command

### Two purposes of `mv`:

| Use Case | Example |
|----------|---------|
| **Rename** file | `mv file1.htm file1.html` |
| **Move** file to another location | `mv file100.html /var/tmp/` |

### Moving a file:

```bash
mv file100.html /var/tmp/
```

> 📌 `/var/tmp/` files untouched for 30 days are automatically deleted.

---

## 7. Creating Large Files — `truncate`

### Create a 1GB file:

```bash
truncate -s 1G file_1GB
```

### Check file size (human-readable):

```bash
ls -lh file_1GB
```

**Output:** `-rw-rw-r--. 1 student student 1.0G Aug 26 16:30 file_1GB`

### truncate size units:

| Unit | Meaning |
|------|---------|
| `1K` | 1 Kibibyte (1024 bytes) |
| `1M` | 1 Mebibyte |
| `1G` | 1 Gibibyte |
| `1T` | 1 Tebibyte |

> 📌 Gigabytes (base 10) vs Gibibytes (base 2) — covered in next course.

---

## 8. Copying Files — The `cp` Command

### Copy a file to another location:

```bash
cp file_1GB /usr/share/
```

### Permission denied? Use `sudo`:

```bash
sudo !!          # Runs previous command with sudo
```

**Example:**
```bash
cp file_1GB /usr/share/          # Permission denied
sudo !!                           # sudo cp file_1GB /usr/share/
```

### Copy a directory (requires recursion):

```bash
cp -r /usr/share/doc ~student
```

| Option | Meaning |
|--------|---------|
| `-r` | Recursive (copy directory + all contents) |
| `-R` | Same as `-r` for most commands |

### Special tilde (`~`) usage:

| Syntax | Meaning |
|--------|---------|
| `~` | Current user's home directory |
| `~student` | Home directory of user `student` |

---

## 9. Directory Size — The `du` Command

```bash
du -sh doc
```

| Option | Meaning |
|--------|---------|
| `-s` | Summary (total size, not per file) |
| `-h` | Human-readable (KB, MB, GB) |

**Output example:** `40M    doc`

---

## 10. Deleting Files/Directories — The `rm` Command

### Delete a file:

```bash
rm file1.html
```

### Delete an empty directory:

```bash
rmdir emptydir      # Only works if empty
```

### Delete a directory with contents (recursive):

```bash
rm -r doc
```

### Force delete (DANGEROUS ⚠️):

```bash
rm -rf doc
```

| Option | Meaning | Risk |
|--------|---------|------|
| `-r` | Recursive (delete directory + contents) | Moderate |
| `-f` | Force (no prompts, ignore errors) | **High** |
| `-rf` | Recursive + Force | **Very High** |

> ⚠️ **`rm -rf` is dangerous.** There is no undo. Double-check before executing.

### Interactive mode (safer):

```bash
rm -ri doc     # Ask before deleting each file
```

---

# Complete Command Reference Table

| Command | Purpose | Example |
|---------|---------|---------|
| `mkdir -p` | Create directories | `mkdir -p parent/child` |
| `Esc + .` | Insert last argument | `cd Esc+.` |
| `touch {1..100}.ext` | Create multiple files | `touch file{1..100}.htm` |
| `rename .old .new *` | Batch rename files | `rename .htm .html *` |
| `mv source dest` | Move or rename | `mv file.html /var/tmp/` |
| `truncate -s SIZE` | Create file of specific size | `truncate -s 1G bigfile` |
| `cp source dest` | Copy file | `cp file1 file2` |
| `cp -r source dest` | Copy directory | `cp -r /usr/share/doc ~/` |
| `du -sh dir` | Show directory size | `du -sh doc` |
| `rm file` | Delete file | `rm file1.html` |
| `rm -r dir` | Delete directory | `rm -r doc` |
| `rm -rf dir` | Force delete (dangerous) | `rm -rf doc` |
| `sudo !!` | Re-run last command with sudo | after permission denied |

---

# Productivity Tips Summary

| Tip | Command/Trick |
|-----|---------------|
| Reuse last argument | `Esc + .` or `Alt + .` |
| Create many files | Brace expansion: `{1..100}` |
| Batch rename | `rename .old .new *` |
| Re-run with sudo | `sudo !!` |
| Check directory size | `du -sh` |
| Get a root shell | `sudo -i` |
| Reference another user's home | `~username` |

---

# Key Takeaways

- **Brace expansion** (`{1..100}`) saves massive amounts of typing.
- **`rename`** batch renames files — don't manually rename 100 files.
- **`truncate`** creates files of specific sizes (great for testing).
- **`sudo !!`** re-runs the previous command with `sudo` — a huge time-saver.
- **`cp -r`** required for copying directories.
- **`rm -rf` is dangerous** — always double-check before using.
- **`du -sh`** shows directory sizes without listing individual files.
- **`~student`** = home directory of user `student` (no slash needed).
- **Work smarter, not harder** — learn these tricks and save hours.