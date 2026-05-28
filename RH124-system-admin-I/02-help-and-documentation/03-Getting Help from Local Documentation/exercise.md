# Guided Exercise — Getting Help from Manual Pages (Notes)

---

# 1. Open the Manual for `man`

Command:

```bash id="z1e0x8"
man man
```

### Important detail:

* Header:

```id="01af2h"
MAN(1)
```

### Meaning:

* `(1)` = user commands section

### Quit:

```id="axdf0v"
q
```

---

# 2. Open the Manual for `passwd`

Command:

```bash id="fgd0zv"
man passwd
```

### What it explains:

* How to change passwords
* Syntax:

```id="t83o1k"
passwd [options] [LOGIN]
```

### Important concept:

* Normal users → change only their own password
* Root → change any user's password

---

# 3. Open the Manual for `/etc/passwd`

Command:

```bash id="k7c4kg"
man 5 passwd
```

### Why section 5?

* Section 5 = file formats/configuration files

### What it explains:

* Structure of `/etc/passwd`
* User account information storage

---

# 4. Open the `su` Manual

Command:

```bash id="7xv1p3"
man su
```

### Important information:

* `su` switches user identity
* Default user = `root`

### Login shell option:

```bash id="95k9h0"
su -
```

### Meaning:

* `-` creates a full login shell
* Loads root’s environment properly

---

# 5. Search for ZIP-related Man Pages

Command:

```bash id="m3d76f"
man -k zip
```

### Example results:

* `zipinfo`
* `zipnote`
* `zipsplit`

### Purpose:

* Searches manual page database by keyword

---

# 6. Find Boot Parameter Documentation

Command:

```bash id="jlwm2i"
man -k boot
```

### Important result:

```id="9x0i7z"
bootparam (7)
```

### Meaning:

* Section 7 = miscellaneous/system conventions

---

# 7. Find ext4 Tuning Command

Command:

```bash id="2v4n2j"
man -k ext4
```

### Important result:

```id="r1qv6n"
tune2fs (8)
```

### Meaning:

* Section 8 = system administration commands

---

# 8. Display One-Line Description of a Command

Command:

```bash id="j6g3ql"
man -f w
```

### Example output:

```id="d0u1qo"
w (1) - Show who is logged on and what they are doing.
```

### Purpose:

* Quick description lookup

---

# 9. Open the `vi` Manual

Command:

```bash id="h8g61k"
man vi
```

### Important option found:

```id="j1v0cc"
+[num]
```

### Meaning:

* Opens file with cursor at specified line

---

# 10. Open File at Specific Line in `vi`

Command:

```bash id="8kwxg6"
vi +2 manual
```

### Result:

* Opens `manual` file
* Cursor starts on line 2

### Exit vi:

```id="psk4ki"
:q
```

---

# Important Man Page Sections

| Section | Purpose                          |
| ------- | -------------------------------- |
| 1       | User commands                    |
| 5       | File formats/config files        |
| 7       | Miscellaneous/system conventions |
| 8       | System administration commands   |

---

# Important Commands Summary

| Command          | Purpose                      |
| ---------------- | ---------------------------- |
| `man man`        | Manual for man               |
| `man passwd`     | Password command docs        |
| `man 5 passwd`   | passwd file format docs      |
| `man su`         | User switching docs          |
| `man -k keyword` | Search man pages             |
| `man -f command` | One-line command description |
| `vi +2 file`     | Open file at line 2          |

---

# Key Takeaways

* Linux includes extensive built-in documentation
* Use `man -k` to search by keyword
* Use section numbers to access correct documentation
* `man -f` gives quick summaries
* `vi +number file` opens a file directly at a specific line
* Learn to rely on man pages instead of memorizing every option
