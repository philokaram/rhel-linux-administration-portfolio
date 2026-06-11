# Redirecting Output in Linux — Notes

## 1. What Is Redirection?

Taking what a command would normally show on screen and sending it somewhere else:

- Into a **file**
- Into **another program**
- Into **nothingness** (`/dev/null`)

> 💡 One of the most powerful tools in Linux.

---

## 2. File Descriptors — Numbered Pipes

Every process communicates using **file descriptors**:

| Descriptor | Name | Symbol | Default Source/Destination |
|------------|------|--------|---------------------------|
| `0` | Standard Input (stdin) | `<` | Keyboard |
| `1` | Standard Output (stdout) | `>` or `>>` | Terminal (screen) |
| `2` | Standard Error (stderr) | `2>` | Terminal (screen) |

> By default, **both** stdout and stderr splash right into your terminal.

---

## 3. Output Redirection — `>` vs `>>`

| Operator | Action | Effect |
|----------|--------|--------|
| `>` | Redirect stdout to file | **Overwrites** existing content |
| `>>` | Redirect stdout to file | **Appends** to existing content |

### Example — overwrite:

```bash
w -i > users.txt
```

- Saves output of `w -i` to `users.txt`
- If file exists → **overwritten**

### Example — append:

```bash
lastlog >> users.txt
```

- Adds output of `lastlog` to end of `users.txt`
- Previous content remains

### Demonstration:

```bash
w -i > users.txt      # File now has w -i output
lastlog >> users.txt  # Now has w -i + lastlog
```

---

## 4. Error Redirection — `2>`

### Problem: Errors clutter useful output

```bash
find / -size +10M
```

**Output:** Lots of "Permission denied" messages + a few actual results.

### Solution — redirect errors to a file:

```bash
find / -size +10M 2> error.log
```

- Errors go to `error.log`
- Normal output still appears on screen

### Discard errors entirely — send to `/dev/null`:

```bash
find / -size +10M 2> /dev/null
```

> `/dev/null` = the big black hole of Linux. Anything sent there is **gone forever**.

### Separate normal output and errors:

```bash
find / -size +10M > good_files.txt 2> errors.log
```

| Redirect | Destination |
|----------|-------------|
| `>` | Normal output → `good_files.txt` |
| `2>` | Errors → `errors.log` |

### Combine both stdout and stderr into one file:

```bash
find / -size +10M &> all_output.txt
```

or

```bash
find / -size +10M > all_output.txt 2>&1
```

---

## 5. The Pipe (`|`) — Chaining Commands Together

Take output of one command and hand it directly to another.

### Syntax:

```bash
command1 | command2
```

### Examples:

| Command | Purpose |
|---------|---------|
| `ls -l | head -5` | Show first 5 files |
| `ls -l | less` | Scroll through long output |
| `echo "password" | passwd --stdin user` | Set password non-interactively |

### Real example — setting a password:

```bash
echo "redhat123" | sudo passwd --stdin devops
```

- `echo "redhat123"` produces output
- Pipe (`|`) sends it as **input** to `passwd --stdin`
- Password for `devops` user becomes `redhat123`

> 💡 Pipes let you chain small, simple commands together to solve big problems.

---

## 6. The `tee` Command — Watch and Save Simultaneously

### Problem: You want to see output on screen **and** save it to a file.

### Solution — `tee`:

```bash
w -i | tee users.txt
```

- Output appears on screen
- **And** gets saved to `users.txt`

### Append with `tee -a`:

```bash
lastlog | tee -a users.txt
```

| Option | Action |
|--------|--------|
| `tee file` | Overwrite file |
| `tee -a file` | **A**ppend to file |

> 💡 Perfect when you want to capture results without losing live view in your terminal.

---

## 7. Advanced Trick — `sudo` with Redirection

### The problem:

```bash
sudo echo "hello world" > /etc/fstab
```

**Fails!** Why?

- `sudo echo "hello world"` runs with elevated privileges ✅
- **Redirection (`>`)** runs as the **student user** ❌

### The solution — pipe to `sudo tee`:

```bash
echo "hello world" | sudo tee -a /etc/fstab
```

| Part | Runs as |
|------|---------|
| `echo "hello world"` | student (no sudo needed) |
| `|` pipe | connects them |
| `sudo tee -a /etc/fstab` | root (append to protected file) |

> 💡 This works because `tee` runs with `sudo` and handles the file writing.

---

## 8. Here Documents (`<<`) — Create Files Without an Editor

### Syntax:

```bash
cat > filename << MARKER
text line 1
text line 2
MARKER
```

- `MARKER` can be any word (commonly `EOF` = End Of File)
- Text is taken from command line until `MARKER` appears on its own line

### Example:

```bash
cat > foo << EOF
Hello world.
This is a doc created from the shell using input and output redirection.
All of this text is saved to foo when we have a line that reads EOF.
EOF
```

### Verify:

```bash
cat foo
```

> 💡 Extremely useful in **shell scripting** for automating file creation.

---

# Complete Redirection Reference Table

| Operator | Name | Effect |
|----------|------|--------|
| `>` | stdout redirect (overwrite) | Send normal output to file (replaces) |
| `>>` | stdout redirect (append) | Send normal output to file (adds) |
| `2>` | stderr redirect (overwrite) | Send errors to file (replaces) |
| `2>>` | stderr redirect (append) | Send errors to file (adds) |
| `&>` | Both stdout + stderr | Send everything to one file |
| `2>&1` | stderr to stdout | Send errors where stdout goes |
| `\|` | Pipe | Send output of command1 to command2 |
| `tee` | Split output | Show on screen AND save to file |
| `tee -a` | Split output (append) | Show and append to file |
| `/dev/null` | Black hole | Discard data forever |
| `<< MARKER` | Here document | Create multi-line file from terminal |

---

# Common Redirection Patterns

| Goal | Command |
|------|---------|
| Save command output | `ls > files.txt` |
| Append to file | `date >> log.txt` |
| Discard errors | `find / -name foo 2> /dev/null` |
| See output + save it | `ls -l \| tee listing.txt` |
| Chain commands | `ps aux \| grep ssh` |
| Set password non-interactively | `echo "pass" \| passwd --stdin user` |
| Write to protected file without sudo issues | `echo "text" \| sudo tee -a /etc/protected` |
| Create multi-line file from script | `cat > file << EOF ... EOF` |

---

# File Descriptor Diagram

```
            Keyboard (stdin=0)
                   │
                   ▼
            ┌─────────────┐
            │   COMMAND    │
            └─────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
   stdout=1                stderr=2
   (normal output)         (errors)
        │                     │
        ▼                     ▼
   Terminal               Terminal

With redirection:
   command > file         command 2> error.log
   (stdout → file)        (stderr → file)
```

---

# Key Takeaways

- **File descriptors**: `0`=stdin (keyboard), `1`=stdout (screen), `2`=stderr (screen)
- **`>`** = redirect stdout (overwrite), **`>>`** = redirect stdout (append)
- **`2>`** = redirect stderr (errors)
- **`&>`** or `2>&1` = redirect both stdout and stderr to same place
- **`/dev/null`** = discard data forever (the black hole)
- **Pipe (`|`)** = connect commands: output of one becomes input of another
- **`tee`** = see output on screen AND save to file simultaneously
- **`sudo` + redirection fails** because redirection runs as user, not root — fix with `sudo tee`
- **Here documents (`<< MARKER`)** = create files without an editor (great for scripts)

> 💡 Once you get comfortable with redirection and pipes, you stop feeling like the shell is in charge of you — and you start feeling like **you're in charge of the shell**.