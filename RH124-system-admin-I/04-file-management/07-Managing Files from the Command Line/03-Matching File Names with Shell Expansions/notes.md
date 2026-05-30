# Matching File Names with Shell Expansions — Notes

## 1. Wildcard Patterns — The Basics

### The `*` (asterisk) — matches any text (including none)

```bash
ls b*
```

**Matches:** `bar`, `baz`, `bongle` (any file starting with `b`)

> 💡 Pronounced "star" — `ls b star`

---

### The `?` (question mark) — matches exactly one character

```bash
ls b??      # b + exactly 2 characters
```

**Matches:** `bar`, `baz` (not `bongle` — too long)

| Pattern | Matches | Does NOT match |
|---------|---------|----------------|
| `b*` | `bar`, `baz`, `bongle`, `b` | — |
| `b?` | `b` + 1 char (e.g., `b1`) | `bar` (2 chars after) |
| `b??` | `b` + exactly 2 chars | `b` (too few), `bongle` (too many) |

---

## 2. Character Sets — Square Brackets `[ ]`

### Match any single character from a set:

```bash
cat [Ee]mail
```

**Matches:** `Email` (uppercase E) AND `email` (lowercase e)

---

### Exclusion — `[^ ]` or `[! ]`

Match any character **NOT** in the set:

```bash
rm ba[^xyz]    # Removes bax, bay, baz? Wait — careful!
```

| Pattern | Matches | Excludes |
|---------|---------|----------|
| `ba[xyz]` | `bax`, `bay`, `baz` | everything else |
| `ba[^xyz]` | Everything EXCEPT `bax`, `bay`, `baz` | `bax`, `bay`, `baz` |

> ⚠️ The `^` at the beginning of `[ ]` means **NOT**.

---

## 3. Brace Expansion — `{ }`

### Sequential ranges:

```bash
mkdir -pv rhel{8..10}-isos-x86_64
```

**Creates:**
- `rhel8-isos-x86_64/`
- `rhel9-isos-x86_64/`
- `rhel10-isos-x86_64/`

### Syntax:

| Pattern | Result |
|---------|--------|
| `{1..10}` | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 |
| `{a..z}` | a, b, c, ..., z |
| `{A..Z}` | A, B, C, ..., Z |
| `{1..10..2}` | 1, 3, 5, 7, 9 (step of 2) |

---

## 4. Command Substitution — `$(command)`

### Run a command and use its output:

```bash
echo "Today is $(date +%A)"
```

**Output:** `Today is Tuesday`

### In a script (`dow.sh`):

```bash
#!/bin/bash
echo "Today is $(date +%A)"
```

### Run the script:

```bash
./dow.sh      # Need ./ because current dir not in $PATH
```

or

```bash
~/dow.sh      # Absolute/relative path from home
```

---

## 5. The `$PATH` Variable

### Why `dow.sh` doesn't run without `./`:

```bash
echo $PATH
```

**Output example:**
```
/home/student/.local/bin:/home/student/bin:/usr/local/bin:/usr/bin:/bin
```

### How command lookup works:

1. Check each directory in `$PATH` (colon-separated)
2. First match wins
3. If not found → `command not found`

> 💡 Current directory (`.`) is **NOT** in `$PATH` by default — security feature.

### To run a command in current directory:

```bash
./command      # Explicitly specify "here"
```

---

## 6. Environment Variables — `env`

### View all environment variables:

```bash
env
```

### Access a variable:

```bash
echo $USER
```

**Output:** `student`

---

## 7. Variable Protection — Curly Braces `{ }`

### The problem: ambiguous variable names

```bash
touch $USER_access.log
```

**Problem:** Shell looks for variable named `USER_` (not found → empty)

### The solution: curly braces `{ }`

```bash
touch ${USER}_access.log
```

**Creates:** `student_access.log`

### Why?

| Without `{}` | With `{}` |
|--------------|-----------|
| `$USER_access` → variable `USER_` | `${USER}` → variable `USER` |
| Then literal `access.log` | Then literal `_access.log` |

---

## 8. Unsetting Variables — `unset`

```bash
unset VARIABLE_NAME
```

**Example:**

```bash
TODAY="Monday"
echo $TODAY      # Monday
unset TODAY
echo $TODAY      # (nothing)
```

---

## 9. Quoting — Escaping Special Characters

### The problem: `$` has special meaning (variable expansion)

| Quote Type | Behavior | Example | Output |
|------------|----------|---------|--------|
| No quotes | Normal expansion | `echo $USER` | `student` |
| Double `"` | Allows expansion | `echo "Value: $USER"` | `Value: student` |
| Single `'` | **Prevents** expansion | `echo 'Value: $USER'` | `Value: $USER` |
| Backslash `\` | Escapes one character | `echo "Value: \$USER"` | `Value: $USER` |

### Examples:

```bash
echo "The value of the variable $USER is $USER"
# Output: The value of the variable student is student

echo "The value of the variable \$USER is $USER"
# Output: The value of the variable $USER is student

echo 'The value of the variable $USER is $USER'
# Output: The value of the variable $USER is $USER
```

### Quick reference:

| You want... | Use... |
|-------------|--------|
| Variable to expand | `"double quotes"` or no quotes |
| Literal `$` symbol | `'single quotes'` or `\$` |
| Literal everything | `'single quotes'` |

---

# Complete Command Reference Table

| Pattern/Command | Purpose | Example |
|-----------------|---------|---------|
| `*` | Any text | `ls b*` |
| `?` | Exactly one character | `ls b??` |
| `[abc]` | One character from set | `cat [Ee]mail` |
| `[^abc]` | One character NOT in set | `rm ba[^xyz]` |
| `{1..10}` | Brace expansion (sequence) | `mkdir dir{1..10}` |
| `$(command)` | Command substitution | `echo $(date)` |
| `./script` | Run script in current dir | `./dow.sh` |
| `echo $PATH` | View command search paths | `echo $PATH` |
| `echo $VAR` | Print variable value | `echo $USER` |
| `${VAR}` | Protected variable | `touch ${USER}_file` |
| `unset VAR` | Delete variable | `unset TODAY` |
| `'single quotes'` | Literal text (no expansion) | `echo '$USER'` |
| `"double quotes"` | Allows expansion | `echo "$USER"` |
| `\$` | Escaped dollar sign | `echo "\$USER"` |

---

# Wildcard Cheat Sheet

| Pattern | Matches | Does NOT match |
|---------|---------|----------------|
| `*` | Everything | — |
| `a*` | Starts with `a` | `bat` |
| `*.txt` | Ends with `.txt` | `file.pdf` |
| `??` | Exactly 2 chars | `a` (1 char), `abc` (3 chars) |
| `[abc]` | `a`, `b`, or `c` | `d` |
| `[a-c]` | `a`, `b`, or `c` (range) | `d` |
| `[^abc]` | NOT `a`, `b`, or `c` | `a`, `b`, `c` |
| `file[0-9]` | `file0`...`file9` | `file10` (2 digits) |

---

# Key Takeaways

- **`*`** = any text (zero or more characters)
- **`?`** = exactly one character
- **`[ ]`** = match one character from a set
- **`[^ ]`** = match one character NOT in a set
- **`{ }`** = brace expansion (generate sequences)
- **`$(command)`** = command substitution (use output as text)
- **`$PATH`** controls where the shell looks for commands (`.` not included by default)
- **`${VAR}`** protects variable names from adjacent characters
- **`unset`** removes a variable
- **Single quotes** `' '` = everything literal (no expansion)
- **Double quotes** `" "` = allows variable expansion
- **Backslash `\`** = escape one special character

> 💡 **Work smarter**: Use wildcards and expansions to avoid typing every filename individually.