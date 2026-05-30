## 1. The Problem

- New Linux users feel like they're wandering a **massive library with no map**.
- Solution: Understand **paths** → you'll never get lost.

---

## 2. Two Ways to Point to a File

| Path Type | Definition | Starts With | Analogy |
|-----------|------------|-------------|---------|
| **Absolute path** | Full GPS coordinates | Leading `/` (e.g., `/var/log/messages`) | Full street address |
| **Relative path** | Shortcut from where you are | No leading `/` (e.g., `log/messages`) | "Turn left at the light" |

> 💡 **Tip for scripting:** Always use **absolute paths** — they're safer.
> 💡 **Tip for command line:** Use **relative paths** to save typing.

---

## 3. Essential Navigation Commands

| Command | Purpose | Example |
|---------|---------|---------|
| `cd <path>` | Change directory | `cd /etc/ssh/` |
| `pwd` | Print/ Present Working Directory | `pwd` → `/home/student` |
| `ls` | List directory contents | `ls -l` |
| `tree` | Show directory tree structure | `tree -F .` |

### Trailing slash (`/`) on directories:

- Optional but denotes the item is a **directory**
- Example: `cd /etc/ssh/` (same as `cd /etc/ssh`)

---

## 4. Special Path Shortcuts

| Symbol | Meaning | Example |
|--------|---------|---------|
| `.` (single dot) | Current directory | `tree -F .` |
| `..` (double dot) | One directory level up | `cd ..` |
| `-` (dash) | Previous working directory | `cd -` |
| `~` (tilde) | Your home directory | `cd ~` |
| (nothing) | Just `cd` alone → home directory | `cd` |

### Example: Moving up multiple levels

```bash
cd ../../..   # Moves up 3 levels
```

---

## 5. Creating Directories — The Right Way

### Basic creation (relative paths):

```bash
mkdir Documents
mkdir Documents/Personal
```

### Better way — `-pv` options:

| Option | Meaning |
|--------|---------|
| `-p` | Create parent directories if needed (no error if exist) |
| `-v` | Verbose — show what was created |

```bash
mkdir -pv isos/{rhel,centos,fedora}/x86_64
```

### Brace expansion `{ }`:

- Creates multiple directories from a comma-separated list
- Example above creates:
  - `isos/rhel/x86_64/`
  - `isos/centos/x86_64/`
  - `isos/fedora/x86_64/`

---

## 6. The `touch` Command — Two Purposes

| Scenario | What Happens |
|----------|--------------|
| File **does not exist** | Creates empty file (0 bytes) |
| File **already exists** | Updates timestamp to current date/time |

### Example:

```bash
touch Documents/Personal/passport.jpg   # Creates empty file
sleep 60                                 # Wait 1 minute
touch Documents/Personal/passport.jpg   # Updates timestamp only
```

---

## 7. The `tree` Command with `-F`

```bash
tree -F .
```

- `-F` = Appends `/` to directory names (visual indicator)
- `.` = Current directory

**Output example:**
```
.
├── Documents/
│   └── Personal/
├── isos/
│   ├── centos/
│   │   └── x86_64/
│   ├── fedora/
│   │   └── x86_64/
│   └── rhel/
│       └── x86_64/
```

---

## 8. Listing Files — `ls -lRa`

```bash
ls -lRa
```

| Option | Meaning |
|--------|---------|
| `-l` | Long listing (permissions, size, timestamp) |
| `-R` | Recursive (show subdirectories) |
| `-a` | All files (including hidden files) |

### Hidden files:

- Begin with a leading period (`.`)
- Examples: `.bashrc`, `.ssh/`, `.`, `..`
- Only visible with `ls -a` or by referring directly by name

---

## 9. Linux Case Sensitivity

⚠️ **Linux is case-sensitive for:**

- File names
- Directory names
- Commands
- Options

### Example:

```bash
echo "rdacosta@redhat.com" > email      # lowercase file
echo "rgdacosta@redhat.com" > Email     # uppercase E file
```

Result: Two **different** files.

> ⚠️ Some commands use `-r` (lowercase) vs `-R` (uppercase) for recursion. `ls` requires uppercase `-R`.

---

## 10. Handling Spaces in Names — Three Methods

### Problem: Spaces are argument separators

```bash
mkdir -pv Our trip to Disney World 2025   # WRONG → creates 6 directories!
```

### Solution 1: Double quotes (`" "`)

```bash
mkdir -pv "Our trip to Disney World 2025"
```

### Solution 2: Single quotes (`' '`)

```bash
mkdir -pv 'Our trip to Disney World 2025'
```

> Difference between `"` and `'` will be covered later.

### Solution 3: Backslash escape (`\ `)

```bash
mkdir -pv Our\ trip\ to\ Edinburgh\ 2024   # Works but painful
```

---

## 11. Path Examples — Putting It All Together

### Starting point: `/usr/share/awk`

| Command | Result | Type |
|---------|--------|------|
| `cd /usr/share/awk` | Go to awk directory | Absolute |
| `cd .` | Stay in same directory | Relative |
| `cd ..` | Go to `/usr/share` | Relative |
| `cd -` | Go back to `/usr/share/awk` | Relative |
| `cd ~` | Go to `/home/student` | Relative |
| `cd` | Go to `/home/student` | Relative |
| `cd ../../..` then `cd home/student` | Go to home (the hard way) | Relative |

### The "ridiculous" relative path to home:

```bash
cd ../../../../home/student
```

> 💡 Sometimes absolute paths are faster. Choose wisely.

---

# Complete Command Reference Table

| Command | Purpose |
|---------|---------|
| `cd /absolute/path` | Change to exact location |
| `cd relative/path` | Change from current location |
| `cd ..` | Up one level |
| `cd -` | Previous directory |
| `cd ~` or just `cd` | Home directory |
| `pwd` | Show current directory |
| `ls -lRa` | Long, recursive, all files listing |
| `tree -F .` | Tree view of current directory |
| `mkdir -pv dir1/dir2` | Create directories with parents, verbose |
| `touch filename` | Create file OR update timestamp |
| `echo "text" > file` | Create file with content |

---

# Key Takeaways

- **Absolute paths** = start with `/` → work from anywhere. Use in scripts.
- **Relative paths** = no leading `/` → save typing on command line.
- `.` = current directory. `..` = parent directory. `-` = previous directory. `~` = home.
- `mkdir -pv` = best practice for creating directories.
- `touch` creates **or** updates timestamps — it does NOT empty existing files.
- Linux is **case-sensitive** — `email` and `Email` are different files.
- Spaces in names = use **quotes** (`" "` or `' '`) or **backslash escapes** (`\ `).
- At 3 AM in production, knowing absolute vs relative paths can save your night.