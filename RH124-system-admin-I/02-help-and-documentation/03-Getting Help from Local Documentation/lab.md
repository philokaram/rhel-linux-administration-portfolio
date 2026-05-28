## # Lab: Get Help from Local Documentation — Notes

## 1. Lab Objective

Create a file named `my_task.txt` and append specific command outputs to it using redirection (`>>`).

### Key Redirection Syntax:

```bash
command >> filename
```

- `>>` appends output to a file (creates file if not exists)
- `>` overwrites a file

---

## 2. Redirection Example

```bash
ls >> /tmp/my-file-names
```

Then view the file:

```bash
cat /tmp/my-file-names
```

---

## Task 1: hostname Command — Display All FQDNs

### Step 1: Search for the hostname man page

```bash
man -k hostname
```

Look for: `hostname (1)`

### Step 2: Open the man page

```bash
man hostname
```

### Step 3: Find the option for all FQDNs

Search inside man page (use `/`):

```
/A
```

Result: **`-A, --all-fqdns`**

### Step 4: Run the command and append to file

```bash
hostname -A >> my_task.txt
```

### Step 5: Verify

```bash
cat my_task.txt
```

**Expected output (example):**
```
workstation.lab.example.com
```

---

## Task 2: date Command — Seconds Since Epoch (Jan 1, 1970 to Jan 1, 2025)

### Background: Epoch Time

- **Epoch** = 00:00:00 UTC on January 1, 1970
- `%s` = seconds since Epoch

### Step 1: Browse the date man page

```bash
man date
```

### Step 2: Find relevant options

Search inside man page:

```
/%s
```

Also look for `-d, --date=STRING`

### Step 3: Run the command

```bash
date -d "Jan 1 2025" +%s >> my_task.txt
```

**Explanation:**

| Part | Meaning |
|------|---------|
| `-d "Jan 1 2025"` | Use this date instead of "now" |
| `+%s` | Format: seconds since Epoch |
| `>> my_task.txt` | Append to file |

### Step 4: Verify

```bash
cat my_task.txt
```

**Expected output (example):**
```
workstation.lab.example.com
1735689600
```

### Step 5: Verify your result (optional)

Convert seconds back to a date:

```bash
date --date='@1735689600'
```

**Expected output:**
```
Wed Jan  1 00:00:00 UTC 2025
```

---

## Task 3: getenforce Command — Display SELinux Mode

### Step 1: Search for SELinux-related commands

```bash
man -k selinux
```

Look for: `getenforce (8)`

### Step 2: Run the command and append to file

```bash
getenforce >> my_task.txt
```

### Step 3: Verify

```bash
cat my_task.txt
```

**Expected output (example):**
```
workstation.lab.example.com
1735689600
Enforcing
```

### Possible SELinux modes:

| Mode | Meaning |
|------|---------|
| `Enforcing` | SELinux active, policy enforced |
| `Permissive` | SELinux active, only logging (no blocking) |
| `Disabled` | SELinux turned off |

---

## Task 4: man Command — Find How to Print a Man Page with PostScript

⚠️ **Special instruction:** Append the **command itself**, not the output.

### Step 1: Open the man page for man

```bash
man man
```

### Step 2: Find printing information

Search inside man page:

```
/print
```

or

```
/troff
```

**Found example:**
```
man -t bash | lpr -Pps
```

**Explanation:**

| Part | Meaning |
|------|---------|
| `man -t` | Format manual page for printing (troff/groff) |
| `bash` | Example command (man page to print) |
| `\|` | Pipe output |
| `lpr -Pps` | Send to printer named "ps" |

### Step 3: Append the command (not output) to the file

Use `echo` to add the command as a string:

```bash
echo "man -t bash | lpr -Pps" >> my_task.txt
```

### Step 4: Verify

```bash
cat my_task.txt
```

**Final expected output:**
```
workstation.lab.example.com
1735689600
Enforcing
man -t bash | lpr -Pps
```

---

# Complete Lab Summary

## Final Contents of `my_task.txt`

| Line | Source | Command Used |
|------|--------|--------------|
| 1 | Task 1 | `hostname -A` |
| 2 | Task 2 | `date -d "Jan 1 2025" +%s` |
| 3 | Task 3 | `getenforce` |
| 4 | Task 4 | `echo "man -t bash | lpr -Pps"` |

---

# Command Reference Table

| Task | Command to Append | What It Does |
|------|------------------|--------------|
| 1 | `hostname -A` | Shows all FQDNs of the machine |
| 2 | `date -d "Jan 1 2025" +%s` | Seconds since Epoch (1970-01-01) |
| 3 | `getenforce` | Shows current SELinux mode |
| 4 | `echo "man -t bash | lpr -Pps"` | Adds the print command as text |

---

# Key Takeaways

- Use `man -k keyword` to **find** the right man page.
- Use `man command` to **read** the documentation.
- Use `/keyword` inside a man page to **search**.
- Use `>>` to **append** output to a file.
- Use `echo "command" >> file` to append the **command text itself**.
- Epoch time = seconds since Jan 1, 1970 (`%s`).
- `getenforce` shows SELinux status (Enforcing/Permissive/Disabled).
- `man -t` formats a man page for printing (PostScript/troff/groff).