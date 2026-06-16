# Customizing the Shell Environment — Notes

## 1. Variables — The Basics

### Creating and accessing variables:

```bash
foo=bar
echo $foo
```

### Rules for variable names:

- Can contain: uppercase, lowercase, numbers, underscores (`_`)
- **Cannot begin with a number**
- Common convention: use **uppercase** for variables (they "pop out")

### Example:

```bash
FNAME=Ricardo
SNAME="da Costa"
echo "Hello, my name is $FNAME $SNAME"
```

### When to quote values:

| Value Contains | Use |
|----------------|-----|
| Spaces | `"double quotes"` |
| Special characters | `'single quotes'` or `\` escape |
| Simple text (no spaces) | No quotes needed |

---

## 2. Viewing Variables — `set` and `grep`

### View all variables:

```bash
set
```

### Filter with `grep`:

```bash
set | grep -i "foo|fname|sname"
```

| Option | Meaning |
|--------|---------|
| `-i` | Case-insensitive search |

> 💡 `grep` is your filtering tool — use it to find what you need.

---

## 3. Quoting — Strong vs Weak

| Quote Type | Name | Behavior | Example |
|------------|------|----------|---------|
| `"` (double) | Weak quotes | **Expands** variables | `echo "$USER"` → `student` |
| `'` (single) | Strong quotes | **Prevents** expansion | `echo '$USER'` → `$USER` |
| `\` (backslash) | Escape | Escapes one character | `echo "\$USER"` → `$USER` |

### Example:

```bash
echo "Hello $FNAME"   # Hello Ricardo
echo 'Hello $FNAME'   # Hello $FNAME
```

---

## 4. Unsetting Variables — `unset`

```bash
unset FNAME
echo $FNAME    # (nothing)
```

---

## 5. Protecting Variables — Curly Braces `{}`

### The problem:

```bash
LOG_DIR=/var/log
FILE_PREFIX=system
echo $LOG_DIR/$FILE_PREFIX_status.log
# Output: /var/log/.log  ❌
```

### Why?

- Shell looks for variable `FILE_PREFIX_status` (with underscore)
- Not found → empty → prints `.log`

### The solution — curly braces:

```bash
echo ${LOG_DIR}/${FILE_PREFIX}_status.log
# Output: /var/log/system_status.log  ✅
```

> 💡 Curly braces `{}` define the **boundaries** of the variable name.

---

## 6. Environment Variables — `export`

### Regular variable (only available in current shell):

```bash
APPDIR=/opt/myapp
```

### Environment variable (available to child processes):

```bash
export APPDIR=/opt/myapp
```

### View environment variables:

```bash
env | grep -i appdir
```

> 💡 Use `export` when child processes need access to the variable.

---

## 7. Command Substitution — `$(command)`

### Capture command output into a variable:

```bash
TODAY=$(date +%A)
echo "Hello! Today is $TODAY"
# Output: Hello! Today is Tuesday
```

### Common `date` formats:

| Format | Output Example | Meaning |
|--------|----------------|---------|
| `+%A` | `Tuesday` | Full weekday name |
| `+%F` | `2025-01-15` | Full date (YYYY-MM-DD) |
| `+%s` | `1736956800` | Seconds since epoch |

---

## 8. Math in the Shell — `$(( ))`

```bash
x=10
y=20
sum=$((x + y))
echo $sum    # 30
```

> 💡 Useful for incrementing counters in scripts.

---

## 9. Persistent Configuration — Login Scripts

### Where to set persistent variables/aliases:

| File | Scope | Who manages |
|------|-------|-------------|
| `/etc/bashrc` | System-wide (all users) | Root |
| `~/.bashrc` | Per-user | Each user |

> 💡 Use `~/.bashrc` for your personal customizations.

### Example `~/.bashrc` additions:

```bash
# BEGIN variables
export EDITOR=/usr/bin/vim
export HISTSIZE=10000
export HISTTIMEFORMAT="%F %T "
export LESS=-X
# END variables

# BEGIN aliases
alias ll='ls -l'
alias ..='cd ..'
alias ...='cd ../..'
alias decomment='grep -vE "^(#|$|;|//)"'
# END aliases

# BEGIN history sync
export PROMPT_COMMAND="history -a"
# END history sync
```

---

## 10. History Customization

### `HISTSIZE` — Number of commands to keep in memory:

```bash
export HISTSIZE=10000
```

### `HISTTIMEFORMAT` — Add timestamp to history:

```bash
export HISTTIMEFORMAT="%F %T "
```

### View history with timestamps:

```bash
history
```

**Output example:**
```
501  2025-09-01 14:43:41 crontab -e
508  2025-09-01 15:23:10 man 5 crontab
```

> 💡 Timestamps help with troubleshooting — you can see when commands were run.

### `PROMPT_COMMAND` — Sync history across terminals:

```bash
export PROMPT_COMMAND="history -a"
```

- Synchronizes history so new tabs have access to previous commands
- Enables `Ctrl+R` and `Esc+.` across all terminals

---

## 11. The `LESS` Pager — `-X` Option

### Problem:

- Man pages clear the screen when you quit
- You lose the example you found

### Solution — Set `LESS=-X`:

```bash
export LESS=-X
```

| Option | Meaning |
|--------|---------|
| `-X` | **No clear** on exit — keep the output visible |

### Benefit:

1. Search man page for `/EXAMPLES`
2. Find the example you need
3. Hit `q` to quit
4. **Copy and paste** the example directly from the screen

> 💡 Great for exams — no need to memorize examples.

---

## 12. The `EDITOR` Variable

### Default behavior:

- Many commands use `vi` (vim-minimal) as the default editor
- No syntax highlighting, no line numbers

### Change default editor:

```bash
export EDITOR=/usr/bin/vim
```

### Alternative editor (nano):

```bash
export EDITOR=/usr/bin/nano
```

### Test:

```bash
crontab -e
```

> 💡 Learn `vim` — it's the standard on RHEL, and it's customizable.

### `~/.vimrc` — Customize vim:

```bash
set number          # Show line numbers
set expandtab       # Use spaces instead of tabs
set tabstop=4       # Tab width = 4 spaces
```

---

## 13. Aliases — Work Smarter, Not Harder

### Temporary alias (current session only):

```bash
alias ll='ls -l'
```

### Persistent alias (in `~/.bashrc`):

```bash
alias ..='cd ..'
alias ...='cd ../..'
alias ll='ls -l'
alias l='ls -la'
alias decomment='grep -vE "^(#|$|;|//)"'
```

### Example — `decomment` alias:

```bash
decomment /etc/ssh/ssh_config
```

**Output:** Only non-comment lines (filters out `#`, empty lines, `;`, `//`)

> 💡 Aliases save typing and reduce errors.

### Aliases with arguments (`$1`):

```bash
alias events='oc get events --sort-by=".lastTimestamp" -n $1'
```

- `$1` = first argument passed to the alias
- Use `\$1` in `~/.bashrc` to prevent expansion at load time

---

## 14. Customizing the Prompt — `PS1`

### `PS1` defines your command prompt.

### Default prompt shows:

```
[user@host ~]$
```

### Liquidprompt (advanced prompt tool):

```
github.com/liquidprompt
```

**Features:**
- Shows return code of previous command
- Shows Git branch
- Shows battery status (laptop)
- Shows load average
- Shows user@host with color coding

### Return code of previous command:

```bash
echo $?
```

| Value | Meaning |
|-------|---------|
| `0` | Success |
| Non-zero | Error (specific meaning depends on command) |

### Example — `true` vs `false`:

```bash
true     # Does nothing, successfully
echo $?  # 0

false    # Does nothing, unsuccessfully
echo $?  # 1 (non-zero)
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Create variable | `VAR=value` |
| Access variable | `echo $VAR` |
| Create environment variable | `export VAR=value` |
| View all variables | `set` |
| View environment variables | `env` |
| Filter variables | `set | grep -i pattern` |
| Unset variable | `unset VAR` |
| Protect variable name | `${VAR}_suffix` |
| Command substitution | `$(command)` |
| Math in shell | `$((x + y))` |
| View history | `history` |
| Create alias | `alias name='command'` |
| Remove alias | `unalias name` |
| View return code | `echo $?` |
| Customize prompt | `export PS1=...` |

---

# Login Script Customization — Quick Reference

| Setting | Purpose | Example |
|---------|---------|---------|
| `HISTSIZE` | History size | `10000` |
| `HISTTIMEFORMAT` | Timestamp history | `"%F %T "` |
| `PROMPT_COMMAND` | Sync history | `"history -a"` |
| `LESS=-X` | Keep man pages on screen | `export LESS=-X` |
| `EDITOR` | Default editor | `/usr/bin/vim` |
| `PS1` | Prompt customization | `[\u@\h \W]\$ ` |
| Aliases | Shortcuts | `alias ll='ls -l'` |

---

# Key Takeaways

- **Variables** store values — `VAR=value`, access with `$VAR`.
- **Environment variables** (`export`) are available to child processes.
- **Curly braces** `${VAR}` protect variable names from adjacent characters.
- **Command substitution** `$(command)` captures output into a variable.
- **History** can be enhanced with `HISTSIZE`, `HISTTIMEFORMAT`, and `PROMPT_COMMAND`.
- **`LESS=-X`** keeps man pages on screen after quitting — great for exams.
- **`EDITOR`** sets your preferred text editor for `crontab`, `visudo`, etc.
- **Aliases** save typing and reduce errors — add them to `~/.bashrc`.
- **`~/.bashrc`** is your personal login script — customize it once, benefit forever.
- **`echo $?`** shows the return code of the previous command (0 = success).
- **Liquidprompt** is a powerful prompt customization tool.
- **Work smarter, not harder** — customize your shell to make you more productive.