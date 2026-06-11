# Managing Local Group Accounts — Notes

## 1. Group Information Storage

All local group information is stored in:

```bash
/etc/group
```

> ⚠️ Singular: `/etc/group` (not `/etc/groups`)

### Example entry for `wheel` group:

```bash
grep wheel /etc/group
```

**Output:**
```
wheel:x:10:student
```

### `/etc/group` fields (colon-separated):

| Field | Example | Meaning |
|-------|---------|---------|
| 1 | `wheel` | Group name |
| 2 | `x` | Password placeholder (passwords in `/etc/gshadow`) |
| 3 | `10` | Group ID (GID) |
| 4 | `student` | Member list (comma-separated) |

---

## 2. Three Core Group Management Commands

| Command | Purpose |
|---------|---------|
| `groupadd` | Create a new group |
| `groupmod` | Modify properties of an existing group |
| `groupdel` | Delete a group |

---

## 3. The `groupadd` Command — Create Groups

### Create a new group:

```bash
groupadd engineering
```

### Verify:

```bash
grep engineering /etc/group
```

**Output:**
```
engineering:x:1003:
```

| Part | Meaning |
|------|---------|
| `engineering` | Group name |
| `x` | Password placeholder |
| `1003` | GID (next available) |
| (blank) | No members yet |

### Common `groupadd` options:

| Option | Meaning | Example |
|--------|---------|---------|
| `-g GID` | Specify Group ID | `groupadd -g 2000 engineering` |
| `-r` | Create system group | `groupadd -r sysgroup` |

> 💡 System groups typically use GIDs below 1000.

---

## 4. The `groupmod` Command — Modify Groups

### Rename a group:

```bash
groupmod -n engineers engineering
```

| Option | Meaning |
|--------|---------|
| `-n NEW_NAME` | **N**ew group name |

**Result:** Group `engineering` becomes `engineers` (same GID)

### Verify:

```bash
grep engineers /etc/group
```

**Output:**
```
engineers:x:1003:
```

### Add a user to a supplementary group:

```bash
groupmod -a -U student engineers
```

| Option | Meaning |
|--------|---------|
| `-a` | **A**ppend (don't overwrite) |
| `-U USER` | Add **U**ser(s) to group |

> ⚠️ **Without `-a`**, `groupmod -U` overwrites the entire member list.

### Add multiple users at once (comma-separated):

```bash
groupmod -a -U student,devops,alisson engineers
```

---

## 5. Viewing Group Membership

### Using `groups` command:

```bash
groups student
```

**Output:**
```
student : student wheel engineers
```

### Using `id` command (more detailed):

```bash
id student
```

**Output:**
```
uid=1000(student) gid=1000(student) groups=1000(student),10(wheel),1003(engineers)
```

| Field | Meaning |
|-------|---------|
| `gid` | **Primary** group |
| `groups` | All groups (primary + supplementary) |

---

## 6. The `usermod` Command — Adding Users to Groups

You can also use `usermod` to manage group membership.

### Append user to supplementary group:

```bash
usermod -a -G engineers devops
```

| Option | Meaning |
|--------|---------|
| `-a` | **A**ppend (critical — prevents overwrite) |
| `-G` | Supplementary **G**roups (comma-separated) |
| `-g` | **P**rimary group (lowercase g) |

### ⚠️ The Danger — Forgetting `-a`:

```bash
usermod -G wheel devops    # NO -a flag!
```

**Result:** Overwrites all supplementary groups. User loses all other group memberships (e.g., `engineers`).

### Correct way — add to multiple groups:

```bash
usermod -a -G wheel,engineers,adm devops
```

### Change primary group (lowercase `-g`):

```bash
usermod -g engineers devops
```

---

## 7. Comparison: `groupmod -U` vs `usermod -G`

| Command | Appends by default? | Flag to append |
|---------|--------------------|----------------|
| `groupmod -U` | No (overwrites) | `-a` |
| `usermod -G` | No (overwrites) | `-a` |

> 💡 **Always use `-a`** with both commands unless you intentionally want to replace all supplementary groups.

---

## 8. The `newgrp` Command — Temporary Primary Group Change

### Start a new shell with a different primary group:

```bash
newgrp engineers
```

### Verify:

```bash
id
```

**Output:**
```
uid=1000(student) gid=1003(engineers) groups=1003(engineers),1000(student),10(wheel)
```

| Change | Effect |
|--------|--------|
| Primary group | Changes from `student` (1000) to `engineers` (1003) |
| Supplementary groups | Unchanged |

### Exit temporary shell:

```bash
exit
```

> 💡 `newgrp` is **temporary** — affects only the current shell session.

---

## 9. The `groupdel` Command — Delete Groups

### Delete a group:

```bash
groupdel engineers
```

### Verify deletion:

```bash
grep engineers /etc/group
```

**Output:** (nothing — group is gone)

### ⚠️ Restrictions:

| Cannot delete if... | Why |
|---------------------|-----|
| Group is a user's **primary group** | User must have a primary group |
| Group has members (as primary) | Change user's primary group first |

### Remove a group as primary group first:

```bash
usermod -g student someuser   # Change primary group
groupdel oldgroup             # Now safe to delete
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Create group | `groupadd groupname` |
| Create group with specific GID | `groupadd -g 2000 groupname` |
| Rename group | `groupmod -n newname oldname` |
| Add user to group (append) | `groupmod -a -U user group` |
| Add user to group (usermod) | `usermod -a -G group user` |
| Add user to multiple groups | `usermod -a -G group1,group2 user` |
| Change user's primary group | `usermod -g group user` |
| View user's groups | `groups user` |
| View detailed group info | `id user` |
| Temporarily change primary group | `newgrp groupname` |
| Delete group | `groupdel groupname` |

---

# Group Management Cheat Sheet

| Action | Command | Critical Flag |
|--------|---------|---------------|
| Create | `groupadd eng` | — |
| Rename | `groupmod -n engineers eng` | `-n` |
| Add user (groupmod) | `groupmod -a -U student eng` | `-a` ⚠️ |
| Add user (usermod) | `usermod -a -G eng student` | `-a` ⚠️ |
| Add to multiple | `usermod -a -G eng,wheel,adm student` | `-a` ⚠️ |
| Set primary | `usermod -g eng student` | `-g` (lowercase) |
| Temporary switch | `newgrp eng` | — |
| Delete | `groupdel eng` | — |

---

# Primary vs Supplementary Groups

| | **Primary Group** | **Supplementary Group** |
|--|------------------|------------------------|
| How many? | Exactly 1 | 0 or more |
| Stored in | `/etc/passwd` (field 4) | `/etc/group` (field 4) |
| Files created inherit | This group | Not by default |
| Change with | `usermod -g` | `usermod -a -G` |
| `newgrp` changes | ✅ (temporarily) | ❌ |

---

# Key Takeaways

- **`/etc/group`** stores local group information (colon-delimited: name:password:GID:members)
- **`groupadd`** = create groups
- **`groupmod`** = modify groups (`-n` to rename, `-a -U` to add members)
- **`groupdel`** = delete groups (fails if group is anyone's primary group)
- **`usermod -a -G`** = add user to supplementary groups (always use `-a`!)
- **`groups`** and **`id`** = view group memberships (`id` gives more detail)
- **`newgrp`** = temporarily change primary group for current shell session
- **Without `-a`**, both `groupmod -U` and `usermod -G` **overwrite** all supplementary groups
- **Primary group** = stored in `/etc/passwd`, affects file creation
- **Supplementary groups** = stored in `/etc/group`, provide additional permissions