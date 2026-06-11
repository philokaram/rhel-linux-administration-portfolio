# Users and Groups in Linux — Notes

## 1. How Linux Identifies Components

| Component | Identifier | Nickname |
|-----------|------------|----------|
| Files | Inode | Index Node |
| Processes | PID | Process ID |
| **Users** | **UID** | **User ID** |
| **Groups** | **GID** | **Group ID** |

> 💡 In computing, we identify elements **numerically**.

---

## 2. Types of Groups

| Group Type | Description | Example |
|------------|-------------|---------|
| **Primary Group** | Default group for user. Each user has exactly one. | User `student` → group `student` |
| **Supplementary Group** | Additional groups a user belongs to | `student` also in group `wheel` |

> By default, each user gets their own primary group (same name as username) and is the only member.

---

## 3. Three Classifications of Users

| User Type | UID Range (typical) | Purpose |
|-----------|---------------------|---------|
| **Super User (root)** | `0` | Full system access. Power comes from **UID=0**, not username |
| **System Users** | `1-999` (RHEL 8/9) | Used by services/daemons. Non-interactive. No login shell. |
| **Regular Users** | `1000+` | Human beings. First regular user gets UID `1000`, increments from there |

> ⚠️ **Important:** Root's power comes from **UID 0**, not from being in group 0. Renaming root doesn't remove privileges — any user with UID 0 is root.

---

## 4. The `id` Command — View User Identity

### Basic usage:

```bash
id
```

**Sample output:**
```
uid=1000(student) gid=1000(student) groups=1000(student),10(root)
```

| Field | Meaning |
|-------|---------|
| `uid` | User ID (and username) |
| `gid` | Primary Group ID (and group name) |
| `groups` | All group memberships (primary + supplementary) |

### Check another user:

```bash
id root
```

**Output:**
```
uid=0(root) gid=0(root) groups=0(root)
```

> 💡 The power of root comes from `uid=0`, not the name "root".

---

## 5. File Ownership — Creator Owner

When a user creates a file:

| Attribute | Source |
|-----------|--------|
| **Owning User** | The user's UID |
| **Owning Group** | The user's **primary group** GID |

### Example:

```bash
ls -l
```

**Output:**
```
-rw-r--r--. 1 student student 1024 Aug 26 10:00 myfile.txt
```

| Column | Meaning |
|--------|---------|
| `student` (first) | **Owning user** |
| `student` (second) | **Owning group** |

> 💡 Use the term **"owning user"** and **"owning group"** — not just "user" or "group".

---

## 6. Processes and Permissions

Processes run with the permissions of:
- The user (UID) that started them
- The primary group (GID) of that user

### View processes with user info:

```bash
ps auww
```

The `USER` column shows which UID the process is running as.

---

## 7. Local User Database — `/etc/passwd`

**Not for passwords!** Contains user account information.

### View student user entry:

```bash
grep student /etc/passwd
```

**Output:**
```
student:x:1000:1000:Student User:/home/student:/bin/bash
```

### `/etc/passwd` fields (colon-separated):

| Field | Example | Meaning |
|-------|---------|---------|
| 1 | `student` | Username |
| 2 | `x` | Password placeholder (actual password in `/etc/shadow`) |
| 3 | `1000` | **User ID (UID)** |
| 4 | `1000` | **Primary Group ID (GID)** |
| 5 | `Student User` | GECOS/Comment field (full name, etc.) |
| 6 | `/home/student` | Home directory |
| 7 | `/bin/bash` | Login shell |

### Special shells:

| Shell | Effect |
|-------|--------|
| `/bin/bash` | Normal login, can run commands |
| `/sbin/nologin` | Can authenticate but cannot log in interactively |
| `/bin/false` | Cannot authenticate or log in |

---

## 8. Local Group Database — `/etc/group`

### View student group entries:

```bash
grep student /etc/group
```

**Output:**
```
student:x:1000:
wheel:x:10:student
```

### `/etc/group` fields (colon-separated):

| Field | Example | Meaning |
|-------|---------|---------|
| 1 | `student` | Group name |
| 2 | `x` | Password placeholder |
| 3 | `1000` | **Group ID (GID)** |
| 4 | (blank) | Member list (comma-separated) |

### Why is `student` group empty?

- Primary group for user `student`
- Membership is **implied** by the user's `/etc/passwd` entry (field 4 = GID 1000)
- No need to list members explicitly

### Supplementary group example:

```
wheel:x:10:student
```

- GID `10` = `wheel` group
- `student` is explicitly listed as a member
- Allows `student` to use `sudo`

---

## 9. Important Concepts Summary

| Concept | Definition | Stored In |
|---------|------------|-----------|
| Username | Human-readable name | `/etc/passwd` |
| User ID (UID) | Numeric identifier | `/etc/passwd` |
| Primary Group ID | User's default group | `/etc/passwd` (field 4) |
| Group Name | Human-readable group name | `/etc/group` |
| Group ID (GID) | Numeric group identifier | `/etc/group` |
| Supplementary Groups | Additional group memberships | `/etc/group` (field 4) |

---

# Command Reference Table

| Command | Purpose |
|---------|---------|
| `id` | Show current user UID, GID, and group memberships |
| `id username` | Show info for another user |
| `ps auww` | Show processes with user column |
| `grep user /etc/passwd` | View user account entry |
| `grep group /etc/group` | View group entry |
| `ls -l` | Show file ownership (owning user + owning group) |

---

# File Ownership Flow

```
User creates file
       │
       ▼
Owning User = User's UID
       │
       ▼
Owning Group = User's Primary GID
       │
       ▼
File permissions checked against:
   - Owning user (UID match)
   - Owning group (GID match in user's groups)
   - Others (everyone else)
```

---

# Key Takeaways

- **UID 0** = root (super user). Power comes from UID, not username or group membership.
- **Regular users** start at UID `1000` and increment.
- **System users** (UID `1-999`) run services, usually have no login shell.
- **`/etc/passwd`** = user account info (7 colon-separated fields). **Does NOT store passwords.**
- **`/etc/group`** = group membership info (4 colon-separated fields).
- **Primary group** = user's default group (from `/etc/passwd` field 4).
- **Supplementary groups** = additional group memberships (from `/etc/group` field 4).
- **`id` command** = quick way to see all UID/GID and group memberships.
- **Processes** run with the permissions of the user (UID) and primary group (GID) that started them.
- When you create a file, the **owning user** = your UID, **owning group** = your primary GID.