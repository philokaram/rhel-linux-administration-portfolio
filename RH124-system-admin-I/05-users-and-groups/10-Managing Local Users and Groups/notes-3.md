# Managing Local User Accounts — Notes

## 1. Three Core User Management Commands

| Command | Purpose |
|---------|---------|
| `useradd` | Create a new user |
| `usermod` | Modify properties of an existing user |
| `userdel` | Delete a user |

---

## 2. Default User Creation Files

When you create a user, defaults are read from two files:

| File | Purpose |
|------|---------|
| `/etc/default/useradd` | Basic user defaults (home, shell, skel, mail spool) |
| `/etc/login.defs` | Advanced defaults (UID ranges, password aging, etc.) |

---

## 3. `/etc/default/useradd` — Key Defaults

```bash
cat /etc/default/useradd
```

| Setting | Default Value | Meaning |
|---------|---------------|---------|
| `HOME` | `/home` | Home directories created here |
| `INACTIVE` | (blank) | Account never disabled due to inactivity |
| `EXPIRE` | (blank) | Account never expires |
| `SHELL` | `/bin/bash` | Default login shell |
| `SKEL` | `/etc/skel` | Skeleton directory (copied to new user's home) |
| `CREATE_MAIL_SPOOL` | `yes` | Create mail spool file in `/var/spool/mail/` |

### Private Group Scheme (User Private Group — UPG):

- When you create a user, a group with the **same name** is created
- User is the **only member** of their primary group
- Group ID (GID) = User ID (UID)

---

## 4. `/etc/login.defs` — Important Settings

```bash
vim /etc/login.defs
:set nu
```

| Setting | Default | Line | Meaning |
|---------|---------|------|---------|
| `MAIL_DIR` | `/var/spool/mail` | ~71 | Mail spool location |
| `PASS_MAX_DAYS` | `99999` | ~124 | Maximum password age (days) |
| `PASS_MIN_DAYS` | `0` | ~128 | Minimum days between password changes |
| `PASS_WARN_AGE` | `7` | ~136 | Days warning before password expires |
| `UID_MIN` | `1000` | ~174 | Minimum UID for regular users |
| `UID_MAX` | `60000` | ~177 | Maximum UID for regular users |
| `CREATE_HOME` | `yes` | ~299 | Create home directory by default |

> 💡 **UID ranges:** 0 = root, 1-999 = system users, 1000-60000 = regular users.

---

## 5. The Skeleton Directory — `/etc/skel`

### What it is:

- Template directory copied to every new user's home directory
- Files/directories here automatically appear in new user's home

### Example — add default directories for all new users:

```bash
mkdir /etc/skel/{Documents,Downloads,Pictures}
```

### Example — add a default `.vimrc` for all new users:

```bash
cp ~/.vimrc /etc/skel/
```

### Verification — create a user and check:

```bash
useradd ricardo
ls -la /home/ricardo/
```

**Output includes:** `.bashrc`, `.bash_profile`, `Documents/`, `Downloads/`, `Pictures/`, `.vimrc`

---

## 6. The `useradd` Command — Create Users

### Basic user creation (all defaults):

```bash
useradd ricardo
```

### Custom user creation with options:

```bash
useradd -c "Alisson B" -u 10000 -s /sbin/nologin -d /dev/null alisson
```
`

### Common `useradd` options:

| Option | Meaning | Example |
|---------|---------|---------|
| `-c "COMMENT"` | GECOS/comment field | `-c "Alisson B"` |
| `-u UID` | Specific User ID | `-u 10000` |
| `-s SHELL` | Login shell | `-s /sbin/nologin` |
| `-d HOME` | Home directory path | `-d /dev/null` |
| `-g GROUP` | Primary group | `-g wheel` |
| `-G GROUPS` | Supplementary groups | `-G wheel,adm` |
| `-M` | Do NOT create home directory | `-M` |

### Example user with nologin shell (service account):

```bash
useradd -s /sbin/nologin -d /dev/null alisson
```

**Purpose:** User can authenticate for services (web, FTP, mail) but **cannot get an interactive login shell**.

---

## 7. The `usermod` Command — Modify Existing Users

### Modify GECOS comment field:

```bash
usermod -c "Ricardo da Costa" ricardo
```

### Lock a user account (disable login):

```bash
usermod -L mo
```

### Unlock a user account:

```bash
usermod -U mo
```

### Common `usermod` options:

| Option | Meaning | Example |
|---------|---------|---------|
| `-c "COMMENT"` | Change GECOS field | `-c "New Name"` |
| `-L` | **L**ock user account | `usermod -L username` |
| `-U` | **U**nlock user account | `usermod -U username` |
| `-s SHELL` | Change login shell | `-s /bin/bash` |
| `-g GROUP` | Change primary group | `-g newgroup` |
| `-G GROUPS` | Set supplementary groups | `-G wheel,adm` |
| `-aG GROUPS` | **A**ppend to supplementary groups | `-aG docker` |

> 💡 Use `-aG` (append) to add groups without removing existing ones.

---

## 8. The `userdel` Command — Delete Users

### Delete user (keep home directory):

```bash
userdel ricardo
```

### Delete user AND home directory:

```bash
userdel -r alisson
```

| Option | Effect |
|--------|--------|
| No option | Removes user from `/etc/passwd`, leaves home directory |
| `-r` | Removes user **and** home directory + mail spool |

---

## 9. ⚠️ The Problem with Deleting Users

### Scenario:

1. User `ricardo` (UID 1002) is deleted
2. New user `mo` (UID 1002) is created
3. `ricardo`'s old files still exist with UID 1002

### Result:

```bash
ls -l /home/
```

**Output:**
```
drwx------. 2 1002 1002 4096 Aug 26 10:00 ricardo/
```

- Old files now **owned by UID 1002** (which is now `mo`)
- `mo` has access to `ricardo`'s files → **accidental data leakage**

### Prevention:

| Better Practice | Why |
|-----------------|-----|
| **Lock the account instead of deleting** | Preserves UID assignment, prevents reuse |
| **Use an identity provider (IDM/AD)** | Centralized identity management |
| **Avoid deleting local user accounts** | Only delete if absolutely necessary |

### Lock instead of delete:

```bash
usermod -L ricardo        # User cannot log in
# Account still exists, UID still reserved
```

---

## 10. Enterprise Best Practices

| Problem | Solution |
|---------|----------|
| Managing users on many systems | Centralized identity provider (IDM, Active Directory) |
| Local user deletion | **Don't delete** — lock instead (`usermod -L`) |
| Home directory cleanup | Archive separately, don't rely on `userdel -r` |
| Service accounts | Use `-s /sbin/nologin` to prevent shell access |

> 💡 **Red Hat IDM** (Identity Management) is included with your RHEL subscription — no extra cost. Covers RH362 course.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Create user (defaults) | `useradd username` |
| Create user with comment | `useradd -c "Full Name" username` |
| Create user with specific UID | `useradd -u 2000 username` |
| Create user with nologin shell | `useradd -s /sbin/nologin username` |
| Create user without home dir | `useradd -M username` |
| Modify user comment | `usermod -c "New Name" username` |
| Lock user account | `usermod -L username` |
| Unlock user account | `usermod -U username` |
| Delete user (keep home) | `userdel username` |
| Delete user + home | `userdel -r username` |
| View user defaults | `cat /etc/default/useradd` |
| View advanced defaults | `cat /etc/login.defs` |
| Populate skeleton dir | `mkdir /etc/skel/{dir1,dir2}` |

---

# User Creation Process Flow

```
useradd username
       │
       ▼
Read /etc/default/useradd
       │
       ▼
Read /etc/login.defs
       │
       ▼
Create /etc/passwd entry (UID, GID, home, shell)
       │
       ▼
Create /etc/group entry (private group)
       │
       ▼
Create /var/spool/mail/username
       │
       ▼
Copy /etc/skel/* → /home/username/
       │
       ▼
Set ownership: chown -R username:username /home/username
```

---

# Key Takeaways

- **`useradd`** = create user (reads defaults from `/etc/default/useradd` and `/etc/login.defs`)
- **`usermod`** = modify existing user (`-c` for comment, `-L` to lock, `-U` to unlock)
- **`userdel`** = delete user (without `-r` keeps home directory, with `-r` deletes it)
- **Skeleton directory** (`/etc/skel`) = template for new user home directories
- **Private group scheme** = each user gets their own primary group (same name, same UID/GID)
- **UID ranges:** 0=root, 1-999=system, 1000-60000=regular users
- **Lock don't delete** — prevents UID reuse and accidental data leakage
- **`/sbin/nologin`** shell = user can authenticate for services but cannot get interactive shell
- **Enterprise solution:** Use centralized identity provider (IDM, AD) instead of managing local users on each system