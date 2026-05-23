---

# 1. Display current date and time

```bash
date
```

---

# 2. Display current time in 24-hour format

```bash
date +%R
```

---

# 3. Identify file type of `/home/student/zcat`

```bash
file zcat
```

### Meaning:

* Output shows it is a **Bash shell script (ASCII text)**
* Yes → it is **human readable**

---

# 4. Display size of the file using `wc` + history shortcut

```bash
wc Esc+.
```

After pressing **Esc + .**, it expands to:

```bash
wc zcat
```

Example output:

```
51 298 1977 zcat
```

---

# 5. Show first 10 lines of file

```bash
head Esc+.
```

Expands to:

```bash
head zcat
```

---

# 6. Show last 10 lines of file

```bash
tail zcat
```

---

# 7. Repeat previous command (2 ways)

### Method 1 (best)

* Press **Up Arrow**
* Press **Enter**

### Method 2:

```bash
!!
```

---

# 8. Show last 20 lines using editing shortcuts

Start with previous command:

```bash
tail zcat
```

Then:

* Press **Up Arrow**
* Press **Ctrl + A** (move to beginning)
* Press **Ctrl + →** (jump word by word)
* Edit command to:

```bash
tail -n 20 zcat
```

Result:

```
(last 20 lines of file)
```

---

# 9. Run `date +%R` again using history

### Step 1:

```bash
history
```

Example:

```
2  date +%R
```

### Step 2:

Run using history number:

```bash
!2
```

Result:

```bash
date +%R
15:59
```

---

# Key Shortcuts Used

| Shortcut   | Purpose                         |
| ---------- | ------------------------------- |
| `Esc + .`  | Reuse last argument             |
| `!!`       | Repeat last command             |
| `Up Arrow` | Scroll command history          |
| `Ctrl + A` | Move to start of line           |
| `Ctrl + →` | Move by word                    |
| `!n`       | Run command from history number |

---
