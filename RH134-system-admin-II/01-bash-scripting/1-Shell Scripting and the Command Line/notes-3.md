# Running Loops and Conditional Commands — Notes

## 1. The Golden Rule

> **"If you can enter it at the command line, you can put it in a shell script."**

---

## 2. If Statements — The Basics

### Syntax (one-liner):

```bash
if [ condition ]; then command1; else command2; fi
```

### Example:

```bash
if [ -d foo ]; then echo "foo exists as a directory"; else echo "foo does not exist as a directory"; fi
```

### Syntax (multi-line — script style):

```bash
if [ -d foo ]; then
    echo "foo exists as a directory"
else
    echo "foo does not exist as a directory"
fi
```

### The `test` command:

- `[ condition ]` is a shortcut for the `test` command
- Use `man test` to see all available conditions

### Common test conditions:

| Condition | Meaning |
|-----------|---------|
| `-d FILE` | File exists and is a directory |
| `-f FILE` | File exists and is a regular file |
| `-e FILE` | File exists (any type) |
| `-z STRING` | String is empty |
| `-n STRING` | String is not empty |
| `STRING1 = STRING2` | Strings are equal |
| `INT1 -eq INT2` | Integers are equal |
| `INT1 -gt INT2` | Greater than |
| `INT1 -lt INT2` | Less than |
| `!` | Negation (NOT) |

---

## 3. The `!!` Trick — Global Substitution

### Problem:

You want to run the previous command but replace all occurrences of `foo` with `bar`.

### Solution:

```bash
!!:gs/foo/bar
```

| Part | Meaning |
|------|---------|
| `!!` | Previous command |
| `:gs` | Global substitution |
| `/foo/` | What to find |
| `/bar/` | What to replace with |
| `/` | Terminator |

> 💡 `g` = global (replace all matches, not just the first).

---

## 4. Functions — Grouping Commands

### Syntax:

```bash
function_name() {
    commands
}
```

### Example — `mcd` (make and change directory):

```bash
mcd() {
    mkdir -pv "$1"
    cd "$1"
}
```

### Use:

```bash
mcd baz
pwd
```

**Output:** `/home/student/baz`

### Check if something is a function:

```bash
type mcd
```

**Output:** `mcd is a function`

> 💡 Functions are great for commands you use repeatedly — add them to `~/.bashrc` for persistence.

---

## 5. For Loops

### Syntax:

```bash
for variable in list; do commands; done
```

### Example — create 9 files:

```bash
for i in {1..9}; do touch file$i; file file$i; done
```

### Multi-line script style:

```bash
for i in {1..9}; do
    touch file$i
    file file$i
done
```

### Brace expansion ranges:

| Pattern | Result |
|---------|--------|
| `{1..9}` | 1, 2, 3, 4, 5, 6, 7, 8, 9 |
| `{1..9..2}` | 1, 3, 5, 7, 9 |
| `{a..z}` | a, b, c, ..., z |
| `{A..Z}` | A, B, C, ..., Z |

---

## 6. Until Loops

### Syntax:

```bash
until [ condition ]; do commands; done
```

### Example — count from 1 to 9:

```bash
COUNT=1
until [ $COUNT -gt 9 ]; do
    echo "The count is at $COUNT"
    ((COUNT++))
    sleep 1
done
```

### How it works:

- Commands run **until** the condition becomes `true`
- When condition is `false` → run
- When condition is `true` → stop

> 💡 `((COUNT++))` increments the variable by 1 (arithmetic expansion).

---

## 7. While Loops

### Syntax:

```bash
while [ condition ]; do commands; done
```

### Example — count from 1 to 9:

```bash
COUNT=1
while [ $COUNT -lt 10 ]; do
    echo "The count is at $COUNT"
    ((COUNT++))
    sleep 1
done
```

### How it works:

- Commands run **while** the condition is `true`
- When condition is `true` → run
- When condition is `false` → stop

### Infinite loop:

```bash
while true; do
    echo "Press Ctrl+C to exit"
    sleep 5
done
```

---

## 8. Read — Getting User Input

### Syntax:

```bash
read -p "Prompt message: " variable_name
```

### Example:

```bash
read -p "Enter your name: " USERNAME
echo "Hello, $USERNAME"
```

### In a script:

```bash
read -p "Enter the full path to the directory to backup: " DIR
echo "You want to backup: $DIR"
```

> 💡 `read` is the primary way to get interactive user input in shell scripts.

---

## 9. Case Statements — Multi-Branch If

### Syntax:

```bash
case $VARIABLE in
    pattern1)
        commands
        ;;
    pattern2)
        commands
        ;;
    *)
        commands
        ;;
esac
```

### Example:

```bash
case $CHOICE in
    1)
        echo "You chose Backup"
        ;;
    2)
        echo "You chose List"
        ;;
    3)
        echo "Exiting..."
        exit 0
        ;;
    *)
        echo "Invalid option"
        ;;
esac
```

> 💡 `;;` terminates each case, `esac` closes the case statement (case backwards).

---

## 10. Basename — Extracting the Last Part of a Path

### Syntax:

```bash
basename /path/to/directory
```

### Example:

```bash
basename /usr/share/doc
```

**Output:** `doc`

### Use case in scripts:

```bash
DIR_NAME=$(basename "$DIR")
BACKUP_FILE="${BACKUP_DIR}/${DIR_NAME}_${TIMESTAMP}.tar.xz"
```

---

## 11. Putting It All Together — Backup Script Example

### Full script structure:

```bash
#!/bin/bash
# Variables
BACKUP_DIR="${HOME}/backups"
TIMESTAMP=$(date +%F_%H:%M)

# Function: Print menu
print_menu() {
    echo "=== Backup Menu ==="
    echo "1. Backup a directory"
    echo "2. List backups"
    echo "3. Exit"
    echo "==================="
}

# Function: Perform backup
backup_directory() {
    read -p "Enter the full path to the directory to backup: " DIR
    if [ ! -d "$DIR" ]; then
        echo "ERROR: ${DIR} does not exist as a directory"
        return 1
    fi
    DIR_NAME=$(basename "$DIR")
    BACKUP_FILE="${BACKUP_DIR}/${DIR_NAME}_${TIMESTAMP}.tar.xz"
    echo "Backing up ${DIR} to ${BACKUP_FILE}"
    tar cfJ "$BACKUP_FILE" "$DIR" 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "Backup successful"
    else
        echo "Backup failed"
    fi
}

# Function: List backups
list_backups() {
    echo "Existing backups in ${BACKUP_DIR}:"
    ls -l "${BACKUP_DIR}"
}

# Main loop
mkdir -p "${BACKUP_DIR}"
while true; do
    print_menu
    read -p "Enter your choice (1, 2, or 3): " CHOICE
    case $CHOICE in
        1) backup_directory ;;
        2) list_backups ;;
        3) echo "Exiting..."; exit 0 ;;
        *) echo "Invalid option. Please choose 1, 2, or 3." ;;
    esac
done
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| If statement (one-liner) | `if [ condition ]; then command; else command; fi` |
| Test if directory exists | `[ -d /path ]` |
| Test if file exists | `[ -f /path ]` |
| Negation (NOT) | `[ ! -d /path ]` |
| Global substitution in previous command | `!!:gs/old/new` |
| Define function | `mcd() { commands; }` |
| For loop | `for i in {1..9}; do command; done` |
| Until loop | `until [ condition ]; do command; done` |
| While loop | `while [ condition ]; do command; done` |
| Infinite loop | `while true; do command; done` |
| Read user input | `read -p "Prompt: " variable` |
| Case statement | `case $VAR in pattern) command;; esac` |
| Basename | `basename /path/to/dir` |
| Arithmetic increment | `((COUNT++))` |

---

# Loop Comparison Table

| Loop Type | Runs | Condition Check |
|-----------|------|-----------------|
| `for` | For each item in list | Before each iteration |
| `while` | While condition is TRUE | Before each iteration |
| `until` | Until condition is TRUE (runs while FALSE) | Before each iteration |

---

# Key Takeaways

- **If statements** test conditions — `[ -d ]` for directories, `[ -f ]` for files.
- **`!!:gs/foo/bar`** is a powerful trick to replace all occurrences of a word in the previous command.
- **Functions** group commands — use them for reusability and clarity.
- **For loops** iterate over a list — great for batch operations.
- **While loops** run while a condition is true.
- **Until loops** run until a condition becomes true (i.e., while false).
- **`read`** gets user input interactively.
- **Case statements** are more readable than multiple `if`/`elif`/`else` chains.
- **`basename`** extracts the last part of a path (useful for creating backup filenames).
- **Infinite loops** (`while true`) are common for menu-driven scripts.
- **Keep it simple** — exam-level shell scripting focuses on basics, not complex functions.