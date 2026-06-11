# Managing User Passwords — Notes

## 1. Where Passwords Are Stored — `/etc/shadow`

| File | Purpose | Permissions |
|------|---------|-------------|
| `/etc/passwd` | User account information | Readable by all |
| `/etc/group` | Group information | Readable by all |
| **`/etc/shadow`** | **Password hashes & aging info** | **Readable only by root** |

> ⚠️ `/etc/shadow` contains sensitive information — normal users cannot view it.

---

## 2. Understanding `/etc/shadow` Format

View entry for a user:

```bash
sudo grep devops /etc/shadow
```

**Example output:**
```
devops:$y$j9T$random_salt$hash_value:20327:2:60:9:5:20453:
```

### Fields (colon-delimited):

| Field | Example | Meaning |
|-------|---------|---------|
| 1 | `devops` | Username |
| 2 | `$y$j9T$...` | Password hash (see breakdown below) |
| 3 | `20327` | Days since epoch (1970-01-01) password was last changed |
| 4 | `2` | **Min days** before password can be changed |
| 5 | `60` | **Max days** password is valid (expires after) |
| 6 | `9` | **Warning days** before expiration |
| 7 | `5` | **Inactivity days** (can log in with expired password) |
| 8 | `20453` | Account **expiration date** (days since epoch) |
| 9 | (blank) | Reserved for future use |

---

## 3. Password Hash Breakdown (Field 2)

The password hash contains multiple components separated by `$`:

```
$y$j9T$random_salt$actual_hash
│ │ │        │
│ │ │        └── 3. Salt (randomized, makes password unique)
│ │ └────────── 2. Options/Parameters (algorithm-specific)
│ └──────────── 1. Hashing algorithm identifier
└────────────── Dollar sign separator
```

### Hashing algorithm identifiers:

| Identifier | Algorithm | Used In |
|------------|-----------|---------|
| `$6$` | SHA-512 | RHEL 7, 8, 9 |
| `$y$` | **Yescrypt** | **RHEL 10** (default, stronger) |

> 💡 **Important:** Passwords are **hashed**, not encrypted. Hashing is one-way; encryption implies decryption with keys.

---

## 4. The `chage` Command — Change Password Aging

### View current password aging settings:

```bash
chage -l devops
```

### Set password aging interactively:

```bash
chage devops
```

### Set password aging with options:

```bash
chage -M 60 -m 2 -W 9 -I 5 -E 2025-12-31 devops
```

### Common `chage` options:

| Option | Meaning | Example |
|--------|---------|---------|
| `-l` | **L**ist current settings | `chage -l username` |
| `-M DAYS` | **M**aximum password age | `-M 60` |
| `-m DAYS` | **M**inimum password age | `-m 2` |
| `-W DAYS` | **W**arning period | `-W 9` |
| `-I DAYS` | **I**nactivity period | `-I 5` |
| `-E DATE` | Account **E**xpiration date | `-E 2025-12-31` |
| `-d 0` | Force password change on next login | `-d 0` |

---

## 5. Locking and Unlocking User Accounts

### Lock a user account (`usermod -L`):

```bash
sudo usermod -L devops
```

**Effect on `/etc/shadow`:** An `!` (exclamation mark) is added before the password hash.

```bash
sudo grep devops /etc/shadow
# Before: $y$j9T$...
# After:  !$y$j9T$...   (locked)
```

### Verify lock status:

```bash
passwd -S devops
```

**Output:** `devops LK 2025-01-15 2 60 9 5` (LK = locked)

### Unlock a user account (`usermod -U`):

```bash
sudo usermod -U devops
```

**Effect:** `!` is removed from `/etc/shadow`

### Verify unlock:

```bash
passwd -S devops
```

**Output:** `devops PS 2025-01-15 2 60 9 5` (PS = password set)

> 💡 **Best practice:** Lock users instead of deleting them to preserve UID and prevent data leakage.

---

## 6. Force Password Change on Next Login

### Method 1 — `chage -d 0`:

```bash
sudo chage -d 0 devops
```

| Part | Meaning |
|------|---------|
| `-d 0` | Set last password change to **day 0** (epoch) |
| Effect | System sees password as **expired** |

### Method 2 — `passwd -e`:

```bash
sudo passwd -e devops
```

| Option | Meaning |
|--------|---------|
| `-e` | **E**xpire password immediately |

### Result:

When `devops` logs in next:
1. Prompted for current password
2. Forced to **immediately change** password
3. Then granted access

> 💡 Perfect for: resetting forgotten passwords, first-time logins, security compliance.

---

## 7. The `passwd` Command — Additional Options

| Command | Effect |
|---------|--------|
| `passwd username` | Change user's password (as root) |
| `passwd` | Change your own password |
| `passwd -l username` | Lock user (same as `usermod -L`) |
| `passwd -u username` | Unlock user (same as `usermod -U`) |
| `passwd -e username` | Expire password (force change on next login) |
| `passwd -S username` | Show password **s**tatus |
| `passwd -d username` | Delete password (dangerous — no password login) |

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View shadow entry | `sudo grep user /etc/shadow` |
| List password aging | `chage -l username` |
| Set max password age | `chage -M 60 username` |
| Set min password age | `chage -m 2 username` |
| Set warning period | `chage -W 9 username` |
| Set inactivity period | `chage -I 5 username` |
| Set account expiration | `chage -E 2025-12-31 username` |
| Force password change | `chage -d 0 username` or `passwd -e username` |
| Lock user account | `usermod -L username` or `passwd -l username` |
| Unlock user account | `usermod -U username` or `passwd -u username` |
| Show password status | `passwd -S username` |

---

# Password Aging Summary

```
Password set
     │
     ▼
[─── MIN DAYS ───]     Cannot change password
     │
     ▼
[─────── MAX DAYS ───────]  Password is valid
     │                         │
     │                    [WARNING] 7 days before expiration
     │                         │
     ▼                         ▼
Password expires ─────► [INACTIVITY]  Can still log in with expired password
                              │
                              ▼
                        Account locked
```

### Example values explained:

| Setting | Value | Meaning |
|---------|-------|---------|
| Last changed | 20327 | Days since epoch (e.g., Jan 15, 2025) |
| Min days | 2 | Must wait 2 days before changing password again |
| Max days | 60 | Password expires after 60 days |
| Warning | 9 | Warn user 9 days before expiration |
| Inactivity | 5 | Can log in for 5 days after expiration |
| Account expires | 20453 | Account disabled on this date |

---

# Lock vs Delete — Best Practice

| Action | Command | Effect | Recommendation |
|--------|---------|--------|----------------|
| **Lock** | `usermod -L user` | User can't log in, data preserved, UID reserved | ✅ **Preferred** |
| **Delete** | `userdel user` | User removed, UID can be reused → data leakage risk | ❌ Avoid |

> **Why lock is better:** Prevents UID reuse and accidental data leakage to new users with same UID.

---

# Key Takeaways

- **`/etc/shadow`** stores password hashes and aging information — readable only by root
- Passwords are **hashed** (not encrypted) — one-way function, no decryption keys
- **Yescrypt** (`$y$`) = default hashing algorithm in RHEL 10
- **SHA-512** (`$6$`) = used in RHEL 7, 8, 9
- **`chage`** = change password aging (`-M` max, `-m` min, `-W` warn, `-I` inactive, `-E` expire)
- **`usermod -L`** = lock user (adds `!` to hash in `/etc/shadow`)
- **`usermod -U`** = unlock user (removes `!`)
- **`passwd -S`** = show password status (LK = locked, PS = password set)
- **`chage -d 0`** or **`passwd -e`** = force password change on next login
- **Best practice:** Lock users instead of deleting them to prevent UID reuse and data leakage
- **Learn by doing** — spending time on the keyboard is more effective than memorizing