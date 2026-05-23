---

# Bash Shell – Executing Commands (Advanced Notes)

## 1. Command Structure (Review)

Basic format:

```
command [options] [arguments]
```

* **Command** → program to run
* **Options** → modify behavior (`-` or `--`)
* **Arguments** → target (file, directory, etc.)

---

## 2. Multiple Arguments

Some commands accept multiple targets.

Example:

```
du -sh /home/student /usr/share/doc
```

* Shows disk usage for multiple directories at once.

---

## 3. Using `date` with Format Options

Command:

```
date +%F
```

### Common formats:

* `%F` → full date (YYYY-MM-DD)
* `%A` → day of week
* `%R` → time (HH:MM)
* `%x` → locale-based date format

👉 These format specifiers change output style.

---

## 4. Command Chaining

### A. Unconditional Execution (`;`)

Runs all commands no matter what happens.

```
whoami ; date ; foo ; bar ; uname -r
```

* Each command runs independently.

---

### B. Conditional AND (`&&`)

Runs next command only if previous succeeds.

```
whoami && date && foo && bar
```

* Stops at first failure (e.g., `foo` fails → stops chain)

---

### C. Conditional OR (`||`)

Runs next command only if previous fails.

```
command1 || command2
```

Example behavior:

* If `command1` succeeds → skip `command2`
* If `command1` fails → run `command2`

---

### D. Mixed Logic

You can combine `&&` and `||`:

```
whoami && foo || date
```

Meaning:

* If `whoami` succeeds → run `foo`
* If `foo` fails → run `date`

---

## 5. Multi-line Commands (Backslash `\`)

Used to make long commands readable.

Example:

```
mkdir -pv foo && \
cd foo && \
...
```

### Key idea:

* `\` = continue command on next line
* `>` prompt indicates continuation
* Command only executes when line ends without `\`

---

## 6. Password Management

### Change password:

```
passwd
```

### Root can change another user password:

```
echo "password" | passwd --stdin student
```

---

## 7. File Viewing Commands

### A. `cat`

* Shows entire file at once (top → bottom)

### B. `tac`

* Reverse of cat (bottom → top)

---

### C. `head`

* First 10 lines by default

```
head file
head -2 file
```

---

### D. `tail`

* Last 10 lines by default

```
tail file
tail -2 file
```

### Live monitoring:

```
tail -f /var/log/messages
```

* Keeps file open and updates in real time
* Stop with `Ctrl + C`

---

### E. `less`

* Paginated file viewer
* Better for large files

Controls:

* `Space` → next page
* `b` → previous page
* `/text` → search
* `n` → next match
* `N` → previous match
* `q` → quit

---

## 8. Command History Tools

### A. View history

```
history
```

---

### B. Reverse search

```
Ctrl + R
```

* Searches previous commands
* Press repeatedly to cycle results

---

### C. Run previous command by number

```
!78
```

* Runs command #78 from history

---

### D. Run last command matching text

```
!ssh
```

* Executes most recent command starting with “ssh”

---

## 9. Argument Recall (Very Useful Trick)

### Escape + dot:

```
ESC + .
```

* Reuses last command argument
* Example:

  ```
  mkdir foo
  cd foo   (using ESC + .)
  ```

---

## 10. Cursor Navigation Shortcuts

| Shortcut          | Action                      |
| ----------------- | --------------------------- |
| Ctrl + A          | Move to start of command    |
| Ctrl + E          | Move to end of command      |
| Ctrl + Left/Right | Move by word                |
| Ctrl + U          | Delete from cursor to start |
| Ctrl + K          | Delete from cursor to end   |

---

## 11. Clear Command History

```
history -c
```

* Clears stored command history

---

## 12. Key Takeaways

* `;` → always run all commands
* `&&` → run only if previous succeeds
* `||` → run only if previous fails
* `\` → split long commands across lines
* `less` is best for viewing large files
* `tail -f` is best for logs (real-time monitoring)
* History tools dramatically improve productivity

---
