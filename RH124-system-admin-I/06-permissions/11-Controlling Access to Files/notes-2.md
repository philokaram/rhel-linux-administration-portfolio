# Managing Permissions from the Command Line — Notes

## 1. Creating Test Directory

### Create `/data` directory as root:

```bash
mkdir /data
```

### View default permissions:

```bash
ls -ld /data
```

**Default output:**
```
drwxr-xr-x 2 root root 4096 May 15 10:00 /data
```

| Field | Value | Meaning |
|-------|-------|---------|
| File type | `d` | Directory |
| Owning user | `root` | Creator = owner |
| Owning group | `root` | Creator's primary group |
| User perms | `rwx` | Full control |
| Group perms | `r-x` | Read + execute (can enter, list, but not write) |
| Other perms | `r-x` | Read + execute |

---

## 2. Changing Permissions — `chmod`

### Two ways to use `chmod`:

| Method | Syntax | Example |
|--------|--------|---------|
| **Octal** | `chmod 000 file` | `chmod 755 script.sh` |
| **Symbolic** | `chmod u+rwx file` | `chmod g-w,o-r file` |

> 💡 In this demonstration, octal is preferred.

### Octal structure: `chmod XYZ file`

| Position | Controls |
|----------|----------|
| `X` (first) | Owning user permissions |
| `Y` (second) | Owning group permissions |
| `Z` (third) | Other permissions |

---

## 3. Removing All Permissions

### Command:

```bash
chmod 000 /data
```

### Result:

```bash
ls -ld /data
# d--------- 2 root root 4096 ... /data
```

### Effect:

| User | Can cd /data? | Can ls? | Can create file? |
|------|--------------|---------|------------------|
| root | ✅ (root bypass) | ✅ | ✅ |
| mo | ❌ | ❌ | ❌ |
| alisson | ❌ | ❌ | ❌ |
| alexis | ❌ | ❌ | ❌ |

> ⚠️ **Root is special** — root can access anything regardless of permissions.

---

## 4. Permission Evaluation — Step by Step

### Questions to ask (in order):

1. Is the user the **owning user**?
   - ✅ Yes → Apply `u` permissions → **STOP**
   - ❌ No → Go to question 2

2. Is the user a member of the **owning group**?
   - ✅ Yes → Apply `g` permissions → **STOP**
   - ❌ No → Go to question 3

3. Apply **other** permissions

> 💡 **First match wins.** Once a match is found, evaluation stops completely.

---

## 5. Adding Execute for Other — `001`

### Command:

```bash
chmod 001 /data
```

### Result:

```bash
ls -ld /data
# d--------x 2 root root 4096 ... /data
```

### Effect on user `mo` (not owner, not in group):

| Action | Permission needed | Result |
|--------|------------------|--------|
| `cd /data` | `--x` (execute) | ✅ Works! |
| `ls /data` | `r-x` (read+execute) | ❌ Permission denied |
| Create file | `rwx` | ❌ Permission denied |

---

## 6. Adding Read+Execute for Other — `005`

### Command:

```bash
chmod 005 /data
```

### Result:

```bash
ls -ld /data
# d----r-x--- 2 root root 4096 ... /data
```

### Effect on user `mo`:

| Action | Result |
|--------|--------|
| `cd /data` | ✅ Works |
| `ls /data` | ✅ Works (can see `root.1`) |
| `cat root.1` | ✅ Works (file has read for other) |
| Delete file | ❌ Need write on directory |

---

## 7. Changing Ownership — `chown`

### Change owning user only:

```bash
chown mo /data
```

### Change owning user AND group:

```bash
chown mo:developers /data
```

### Change owning group only:

```bash
chown :developers /data
```

### Verify:

```bash
ls -ld /data
# drwxr-xr-x 2 mo developers 4096 ... /data
```

---

## 8. Critical Concept — Permissions Follow User Identity

### Scenario: `mo` is now owning user of `/data` with perms `000`

```bash
chown mo /data
chmod 000 /data
ls -ld /data
# d--------- 2 mo root 4096 ... /data
```

### Can `mo` access `/data`?

| Question | Answer |
|----------|--------|
| Is `mo` the owning user? | ✅ **YES** |
| Apply owning user permissions | `---` (nothing) |
| **STOP** — group and other are **NOT** checked | |

**Result:** ❌ `mo` cannot access `/data` despite being owner!

> 💡 **This is the most common mistake:** Ownership doesn't grant access — **permissions do**.

---

## 9. Group Membership — Delayed Effect

### Add user to group:

```bash
usermod -aG developers mo
```

### Group membership is NOT active until next login:

```bash
groups          # Doesn't show 'developers' yet
exit            # Log out
ssh servera     # Log back in
groups          # Now shows 'developers'
```

> ⚠️ Group membership changes only take effect at **next login**.

---

## 10. Permission Evaluation — Group Match Example

### Setup:

```bash
chown mo:developers /data
chmod 057 /data
ls -ld /data
# d---r-xrwx 2 mo developers 4096 ... /data
```

| Component | Permissions |
|-----------|-------------|
| Owning user (`mo`) | `---` (0) |
| Owning group (`developers`) | `r-x` (5) |
| Other | `rwx` (7) |

### User `alisson` (member of `developers`):

| Question | Answer | Action |
|----------|--------|--------|
| Is `alisson` the owning user? | ❌ No (mo is) | Continue |
| Is `alisson` in owning group? | ✅ Yes (developers) | Apply group perms (`r-x`) |
| **STOP** — other permissions ignored | | |

**Result:** `alisson` has read+execute (can cd and ls, but not write)

---

## 11. Deleting Files — Directory Permissions Matter

### Critical insight:

> When you delete a file, you are transacting against the **DIRECTORY**, not the file.

### Example: User `alexis` (not owner, not in group)

Directory permissions for `alexis` = other = `rwx`

| Action | Permission needed on | Does `alexis` have it? | Result |
|--------|---------------------|----------------------|--------|
| Delete `root.1` | Write on **directory** | ✅ Yes (other has `w`) | ✅ Can delete |
| Modify content of `root.1` | Write on **file** | ❌ No (file perms restrict) | ❌ Cannot modify |

> 💡 You can delete a file you cannot read or write — if you have write permission on its **directory**.

---

## 12. Symbolic Permissions — Alternative to Octal

### Add permissions:

```bash
chmod u+rwx /data    # Add rwx for owning user
chmod g+rwx /data    # Add rwx for owning group
```

### Remove permissions:

```bash
chmod o-rwx /data    # Remove all permissions for other
```

### Symbolic syntax:

| Symbol | Meaning |
|--------|---------|
| `u` | Owning user |
| `g` | Owning group |
| `o` | Other |
| `a` | All (u+g+o) |
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set exactly (overwrite) |

---

## 13. Final Cleanup — Sensible Permissions

### Goal:

- Owning user = `root` (with `rwx`)
- Owning group = `developers` (with `rwx`)
- Other = no permissions

### Commands:

```bash
chown root:developers /data
chmod u+rwx /data
chmod g+rwx /data
chmod o-rwx /data
```

### Or in one line:

```bash
chown root:developers /data && chmod 770 /data
```

### Verify:

```bash
ls -ld /data
# drwxrwx--- 2 root developers 4096 ... /data
```

---

# Complete Command Reference Table

| Command | Purpose | Example |
|---------|---------|---------|
| `chmod 000 file` | Remove all permissions | `chmod 000 /data` |
| `chmod 001 file` | Add execute for other | `chmod 001 /data` |
| `chmod 005 file` | Add read+execute for other | `chmod 005 /data` |
| `chmod 770 file` | rwx for user+group, none for other | `chmod 770 /data` |
| `chown user file` | Change owning user | `chown mo /data` |
| `chown :group file` | Change owning group | `chown :developers /data` |
| `chown user:group file` | Change both | `chown mo:developers /data` |
| `usermod -aG group user` | Add user to group | `usermod -aG developers mo` |
| `groups` | Show group membership | `groups` |
| `ls -ld directory` | View directory permissions | `ls -ld /data` |

---

# Permission Evaluation Flowchart

```
                    ┌─────────────────┐
                    │ Is user the     │
                    │ owning user?    │
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │ YES                          │ NO
              ▼                              ▼
    ┌─────────────────┐            ┌─────────────────┐
    │ Apply u perms   │            │ Is user in      │
    │ STOP            │            │ owning group?   │
    └─────────────────┘            └────────┬────────┘
                                             │
                              ┌──────────────┴──────────────┐
                              │ YES                          │ NO
                              ▼                              ▼
                    ┌─────────────────┐            ┌─────────────────┐
                    │ Apply g perms   │            │ Apply o perms   │
                    │ STOP            │            │ STOP            │
                    └─────────────────┘            └─────────────────┘
```

---

# Key Takeaways

- **`chmod`** changes permissions (octal or symbolic).
- **`chown`** changes ownership (user and/or group).
- **Permission evaluation stops at first match** (u → g → o).
- **Root is special** — root can access anything regardless of permissions.
- **Ownership does NOT grant access** — permissions do.
- **Group membership changes** require logout/login to take effect.
- **Deleting a file** requires **write permission on the directory**, not the file.
- **Octal quick reference:** `r=4`, `w=2`, `x=1` (add for combinations).
- **Common directory permissions:**
  - `755` = `rwx r-x r-x` (user full, group/other read+execute)
  - `770` = `rwx rwx ---` (user+group full, other nothing)
  - `700` = `rwx --- ---` (user only)
- **When troubleshooting:** Always check **which entity** (u/g/o) applies to the user first.