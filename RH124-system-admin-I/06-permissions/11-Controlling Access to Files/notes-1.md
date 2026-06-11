# Interpreting Linux File Permissions — Notes

## 1. The Big Picture

Linux permissions are **simple** — only 3 basic permissions:

| Permission | Symbol | Octal Value | Files | Directories |
|------------|--------|-------------|-------|-------------|
| Read | `r` | 4 | View content | List contents (`ls`) |
| Write | `w` | 2 | Modify content | Create/delete files |
| Execute | `x` | 1 | Run as program | Enter (`cd`) |

### Octal shorthand:

| Octal | Permissions |
|-------|-------------|
| 7 | `rwx` (read + write + execute) |
| 6 | `rw-` (read + write) |
| 5 | `r-x` (read + execute) |
| 4 | `r--` (read only) |
| 3 | `-wx` (write + execute) |
| 2 | `-w-` (write only) |
| 1 | `--x` (execute only) |
| 0 | `---` (none) |

> 💡 Just remember: **r=4, w=2, x=1**. Add them together for combinations.

---

## 2. Three Permission Components

Permissions are assigned to **three different entities**:

| Component | Who it applies to |
|-----------|-------------------|
| **Owning user** (u) | The user who owns the file |
| **Owning group** (g) | Members of the group that owns the file |
| **Other** (o) | Everyone **NOT** the owning user AND **NOT** in the owning group |

> ⚠️ **Important terminology:** Always say "owning user" and "owning group" — not just "user" or "group" (to avoid confusion with ACL named users/groups).

### Permission evaluation order:

```
1. Is the process running as the owning user? → Apply u permissions
2. No? Is the process running as a member of the owning group? → Apply g permissions
3. No? → Apply o permissions (everyone else)
```

> 💡 Once a match is found, evaluation stops. First match wins.

---

## 3. Viewing Permissions — `ls -l`

```bash
ls -l
```

### The 10 permission bits:

```
-rw-rw-r-- 1 student student 1042 May 15 10:30 file.txt
 ||│││││││
 |│││││││└─── Other: execute (---)
 │││││││└──── Other: write (---)
 ││││││└───── Other: read (r--)
 │││││└────── Owning group: execute (---)
 ││││└─────── Owning group: write (-w-)
 │││└──────── Owning group: read (r--)
 ││└───────── Owning user: execute (---)
 │└────────── Owning user: write (-w-)
 └─────────── Owning user: read (r--)
```

### Bit by bit:

| Position(s) | Meaning |
|-------------|---------|
| Bit 1 (char 1) | File type (`-`=file, `d`=directory, `l`=symlink, `b`=block, `c`=character) |
| Bits 2-4 | Owning user permissions (`rwx`) |
| Bits 5-7 | Owning group permissions (`rwx`) |
| Bits 8-10 | Other permissions (`rwx`) |

---

## 4. Permissions on Files vs Directories — Critical Differences

| Permission | Effect on **Files** | Effect on **Directories** |
|------------|---------------------|---------------------------|
| **Read (r)** | View file content (`cat`, `less`) | List directory contents (`ls`) |
| **Write (w)** | Modify file content (`vim`, `echo >>`) | Create/delete **files inside** directory |
| **Execute (x)** | Run file as a program/script | Enter the directory (`cd`) |

### Directory minimum requirements:

| What you want to do | Minimum permissions needed |
|---------------------|---------------------------|
| Enter a directory (`cd`) | `--x` (execute only) |
| List directory contents (`ls`) | `r-x` (read + execute) |
| Create/delete files inside | `-wx` (write + execute) |
| Full control of directory | `rwx` |

> ⚠️ **Important:** When you create or delete a file, you are transacting against the **directory**. You need **write + execute** on the directory, not necessarily on the file itself.

### File minimum requirements:

| What you want to do | Minimum permissions needed |
|---------------------|---------------------------|
| View file content | `r--` (read) |
| Modify file content | `rw-` (read + write) |
| Run as program | `r-x` (read + execute) |

---

## 5. Examples — Reading Permission Strings

### Example 1: `drwxrwxr-x`

```
d rwx rwx r-x
│ │   │   └── Other: read + execute (can enter, list, but not write)
│ │   └────── Owning group: read + write + execute (full control)
│ └────────── Owning user: read + write + execute (full control)
└──────────── File type: directory
```

### Example 2: `-rw-rw-r--`

```
- rw- rw- r--
│ │   │   └── Other: read only
│ │   └────── Owning group: read + write
│ └────────── Owning user: read + write
└──────────── File type: regular file
```

### Example 3: `dr-xr-x---`

```
d r-x r-x ---
│ │   │   └── Other: no permissions
│ │   └────── Owning group: read + execute (can enter and list, but not write)
│ └────────── Owning user: read + execute (can enter and list, but not write)
└──────────── File type: directory
```

---

## 6. Viewing Directory Permissions — `ls -ld`

### The problem:

```bash
ls -l .              # Shows contents of current directory, not the directory itself
```

### The solution:

```bash
ls -ld .             # Shows the directory's own permissions
```

| Option | Meaning |
|--------|---------|
| `-d` | List directory entry itself, not its contents |

### Example:

```bash
ls -ld /home/student
```

**Output:** `drwx--x--- 2 student student 4096 May 15 10:30 /home/student`

---

## 7. The "Create a Owner" Concept

- When you **create** a file or directory, you become the **owning user**.
- The **owning group** is set to your **primary group**.
- On RHEL: **Private Group Naming Scheme** — each user has a group with the same name.

### Example:

```bash
useradd alice
# Group 'alice' is created automatically
# alice's primary group = alice
```

---

## 8. Permission Evaluation — Why Order Matters

### Scenario: `file.txt` with permissions:

| Component | Permissions |
|-----------|-------------|
| Owning user | `rw-` (read + write) |
| Owning group | `r--` (read only) |
| Other | `---` (none) |

### Who can do what?

| User | Is owning user? | In owning group? | Permission applied | Can write? |
|------|----------------|------------------|--------------------|------------|
| `alice` (owner) | ✅ Yes | N/A | `rw-` | ✅ Yes |
| `bob` (member of owning group) | ❌ No | ✅ Yes | `r--` | ❌ No |
| `charlie` (neither) | ❌ No | ❌ No | `---` | ❌ No |

> 💡 `bob` has execute? No — but that's fine for a file (only matters for directories or executable files).

---

# Complete Permission Reference Table

| Symbol | Permission | File | Directory |
|--------|------------|------|-----------|
| `r` | Read | View content | List contents |
| `w` | Write | Modify content | Create/delete files |
| `x` | Execute | Run as program | Enter (`cd`) |
| `-` | No permission | — | — |

### Octal Values:

| Permission | Octal |
|------------|-------|
| `r` | 4 |
| `w` | 2 |
| `x` | 1 |

### Common Combinations:

| Octal | Symbolic | Meaning |
|-------|----------|---------|
| 7 | `rwx` | Read, write, execute |
| 6 | `rw-` | Read, write |
| 5 | `r-x` | Read, execute |
| 4 | `r--` | Read only |
| 3 | `-wx` | Write, execute |
| 2 | `-w-` | Write only |
| 1 | `--x` | Execute only |
| 0 | `---` | None |

---

# Key Takeaways

- **Only 3 permissions:** read (`4`), write (`2`), execute (`1`).
- **Three components:** owning user (u), owning group (g), other (o).
- **Evaluation order:** u → g → o (first match wins).
- **Files vs Directories:** `x` means "enter" for dirs, "run" for files.
- **To `cd` into a directory:** need `--x` (minimum).
- **To `ls` a directory:** need `r-x`.
- **To create/delete files in a directory:** need `rwx` on the directory.
- **To view a file:** need `r--`.
- **To modify a file:** need `rw-`.
- **To run a file as a program:** need `r-x`.
- Use `ls -l` for files and `ls -ld` for directory permissions.
- Always say **"owning user"** and **"owning group"** to avoid ambiguity.