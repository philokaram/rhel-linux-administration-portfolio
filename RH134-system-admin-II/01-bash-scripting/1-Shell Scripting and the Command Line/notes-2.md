# Writing Basic Bash Scripts — Notes

## 1. What Is a Shell Script?

- A **file** containing a series of commands
- The system executes all commands in the file
- Enables automation and better control over the system

> 💡 **If you can run it from the command line, it can go into your script.**

---

## 2. Script Structure — The Shebang

### Line 1 — The Shebang (`#!`):

```bash
#!/bin/bash
```

| Symbol | Meaning |
|--------|---------|
| `#!` | Shebang — tells the system which interpreter to use |
| `/bin/bash` | The shell that will execute the script |

> 💡 Even if you're a Z shell user, scripts run through the interpreter specified in the shebang.

### Example script — `basic.sh`:

```bash
#!/bin/bash
# Basic system status script
echo "Hello $USER, today is $(date +%A)"
echo -e "\nSystem Status"
echo "Uptime: $(uptime -p)"
echo "Load Average: $(cat /proc/loadavg | awk '{print $1, $2, $3}')"
echo -e "\nUser Status:"
who
echo -e "\nStorage Info:"
lsblk -fs
echo -e "\nMemory Info:"
free -h
echo -e "\nMost CPU Intensive Processes:"
ps -eo pid,comm,%cpu,%mem --sort=-%cpu | head -5
```

> 💡 Use `echo -e` to interpret special characters like `\n` (new line), `\t` (tab).

---

## 3. Making a Script Executable

### Step 1 — Check permissions:

```bash
ls -l basic.sh
```

**Output:** `-rw-r--r--` (no execute permission)

### Step 2 — Add execute permission:

```bash
chmod +x basic.sh
```

### Step 3 — Run the script:

```bash
./basic.sh
```

> ⚠️ The leading `./` is required because the current directory (`.`) is not in `$PATH` by default.

### Why `./` is needed:

- The shell searches for commands in directories listed in `$PATH`
- `.` (current directory) is not in `$PATH` for security reasons
- `./basic.sh` tells the shell: "right here in this directory"

---

## 4. Positional Arguments — `$1`, `$2`, `$#`, `$@`, `$0`

| Variable | Meaning | Example |
|----------|---------|---------|
| `$0` | Script name | `./myscript.sh` |
| `$1` | First argument | `-l` |
| `$2` | Second argument | `/var/log` |
| `$3` | Third argument | `Boston` |
| `$#` | Total number of arguments | `3` |
| `$@` | All arguments (as separate words) | `-l /var/log Boston` |
| `$*` | All arguments (as one string) | `"-l /var/log Boston"` |

### Example script — `args.sh`:

```bash
#!/bin/bash
echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Third argument: $3"
echo "Total arguments: $#"
echo "All arguments: $@"
```

### Run:

```bash
./args.sh /etc/ssh ssh_config.d orange
```

**Output:**
```
Script name: ./args.sh
First argument: /etc/ssh
Second argument: ssh_config.d
Third argument: orange
Total arguments: 3
All arguments: /etc/ssh ssh_config.d orange
```

---

## 5. Return Codes — `$?` and `exit`

- Every command returns a **return code** (exit status)
- `0` = **success**
- Non-zero = **error** (specific meaning depends on the command)

### Check return code:

```bash
echo $?
```

### In scripts:

```bash
ls /some/directory >/dev/null 2>&1
if [ $? -eq 0 ]; then
    echo "Success: Directory exists"
else
    echo "Error: Directory not found"
fi
```

### Force a return code:

```bash
exit 0    # Success
exit 1    # Generic error
```

> 💡 `>/dev/null 2>&1` discards all output (both stdout and stderr).

---

## 6. If Statements — Conditional Logic

### Basic syntax:

```bash
if [ condition ]; then
    # commands when condition is true
else
    # commands when condition is false
fi
```

### Semilcolon (`;`) trick:

The semicolon (`;`) separates commands on the same line:

```bash
if [ $? -eq 0 ]; then
```

### Common test conditions:

| Condition | Meaning |
|-----------|---------|
| `[ $? -eq 0 ]` | Return code equals 0 |
| `[ $? -ne 0 ]` | Return code not equal to 0 |
| `[ -f "$file" ]` | File exists and is a regular file |
| `[ -d "$dir" ]` | Directory exists |
| `[ -z "$var" ]` | Variable is empty |
| `[ -n "$var" ]` | Variable is not empty |

### Example with a script:

```bash
#!/bin/bash
ls "$1" >/dev/null 2>&1
if [ $? -eq 0 ]; then
    echo "Success: Directory $1 exists"
else
    echo "Error: Directory '$1' not found"
fi
```

---

## 7. Arrays — Multiple Values

### Create an array:

```bash
my_array=(mango pineapple plum)
```

### Access array elements:

| Syntax | Meaning |
|--------|---------|
| `${my_array[@]}` | All elements (full array) |
| `${my_array[0]}` | First element (index 0) |
| `${my_array[1]}` | Second element (index 1) |
| `${my_array[*]}` | All elements (as one string) |

### Add an element to an array:

```bash
my_array+=(new_item)
```

### Example:

```bash
#!/bin/bash
my_array=(mango pineapple plum)
echo "Full array: ${my_array[@]}"
echo "First index: ${my_array[0]}"
echo "Second index: ${my_array[1]}"

# Add a new element
my_array+=(grape)
echo "New array: ${my_array[@]}"
```

**Output:**
```
Full array: mango pineapple plum
First index: mango
Second index: pineapple
New array: mango pineapple plum grape
```

---

## 8. Command Substitution — `$(command)`

```bash
TODAY=$(date +%A)
echo "Hello, today is $TODAY"
```

### In scripts:

```bash
#!/bin/bash
echo "Uptime: $(uptime -p)"
echo "Load Average: $(cat /proc/loadavg | awk '{print $1, $2, $3}')"
```

---

## 9. Here Documents (`<< EOF`) — Creating Files from Scripts

A **Here Document** (`<< EOF`) allows you to specify the contents of a file directly in the script.

### Syntax:

```bash
cat << EOF > /path/to/output/file
content line 1
content line 2
$(command substitution works)
EOF
```

### Example — `report.sh`:

```bash
#!/bin/bash
REPORT_FILE=~/system_report.txt

cat << EOF > "$REPORT_FILE"
System Status Report
=====================
Date: $(date)
Uptime: $(uptime -p)
Load Average: $(cat /proc/loadavg | awk '{print $1, $2, $3}')
Users Logged In:
$(who)
Memory Info:
$(free -h)
EOF

echo "System report saved to $REPORT_FILE"
```

### Run:

```bash
chmod +x report.sh
./report.sh
cat ~/system_report.txt
```

> 💡 Here Documents are great for creating configuration files or reports without using multiple `echo` statements.

---

## 10. Script Debugging Tips

### Tip 1 — Copy and paste to the command line:

If a script doesn't work, copy the problematic line and run it directly in the terminal. This gives you immediate error feedback.

### Tip 2 — Use `bash -x` to debug:

```bash
bash -x ./basic.sh
```

| Option | Meaning |
|--------|---------|
| `-x` | Trace mode — shows each command before execution |

### Tip 3 — Add `echo` statements:

```bash
echo "DEBUG: variable value is $VAR"
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Create script | `#!/bin/bash` (first line) |
| Make script executable | `chmod +x script.sh` |
| Run script | `./script.sh` |
| Run with debug | `bash -x script.sh` |
| Script name | `$0` |
| First argument | `$1` |
| All arguments | `$@` |
| Number of arguments | `$#` |
| Last return code | `$?` |
| Force exit code | `exit 0` |
| If statement | `if [ condition ]; then ... fi` |
| Test if directory exists | `[ -d "$dir" ]` |
| Test if file exists | `[ -f "$file" ]` |
| Create array | `arr=(item1 item2)` |
| Access array | `${arr[@]}` |
| Add to array | `arr+=(new_item)` |
| Command substitution | `$(command)` |
| Here Document | `cat << EOF > file ... EOF` |

---

# Special Variables Reference

| Variable | Meaning | Example Value |
|----------|---------|---------------|
| `$0` | Script name | `./myscript.sh` |
| `$1` | First argument | `-l` |
| `$2` | Second argument | `/var/log` |
| `$#` | Number of arguments | `3` |
| `$@` | All arguments (separate) | `-l /var/log Boston` |
| `$*` | All arguments (one string) | `"-l /var/log Boston"` |
| `$?` | Return code of last command | `0` or `1` |
| `$$` | PID of the shell/script | `12345` |
| `$!` | PID of last background command | `12346` |

---

# Key Takeaways

- **Shebang** (`#!/bin/bash`) tells the system which interpreter to use.
- **Make scripts executable** with `chmod +x` before running.
- **`./`** is required because `.` is not in `$PATH` by default.
- **Positional arguments**: `$1`, `$2`, `$3`... `$@` (all), `$#` (count), `$0` (script name).
- **Return codes**: `0` = success, non-zero = error. Check with `$?`.
- **If statements** use `[ condition ]` with spaces inside the brackets.
- **Arrays** store multiple values — indices start at 0.
- **Command substitution** `$(command)` captures output into a variable.
- **Here Documents** (`<< EOF`) allow you to generate entire files from within a script.
- **Debugging tip**: Copy and paste problematic lines to the command line to see errors directly.
- **`bash -x`** enables trace mode — shows each command as it executes.