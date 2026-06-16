# Managing SSH Host Keys — Notes

## 1. SSH Host Keys — The Concept

When you connect to a remote system via SSH, the server presents a **public host key**.

- Like the server's **fingerprint**
- Your SSH client compares it to what it already knows

### Two locations for known hosts:

| File | Scope | Location |
|------|-------|----------|
| User `known_hosts` | Per-user | `~/.ssh/known_hosts` |
| System `known_hosts` | System-wide (all users) | `/etc/ssh/ssh_known_hosts` |

### What `known_hosts` stores:

Each line contains:
- Host name **or** IP address
- Host key fingerprint

---

## 2. The SSH Connection Flow

```
Client connects to servera
          │
          ▼
Server presents host key
          │
          ▼
Client checks known_hosts
          │
          ├── Key found? → Connection proceeds
          │
          └── Key not found → Prompt: "Are you sure you want to continue?"
```

### Prompt example:

```
The authenticity of host 'servera (172.25.250.10)' can't be established.
ECDSA key fingerprint is SHA256:abc123...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

---

## 3. StrictHostKeyChecking — Controlling the Behavior

### Defined in SSH client configuration:

| Value | Behavior |
|-------|----------|
| `ask` | **(Default)** Prompt user for confirmation |
| `yes` | **Maximum security** — never accept new keys automatically; connection fails if key not found |
| `no` | **Convenience over security** — automatically accept any key (dangerous) |
| `accept-new` | **Middle ground** — new hosts added automatically, but changed keys rejected |

### Configuration hierarchy (precedence):

1. **Command-line options** (strongest)
2. **User config file** (`~/.ssh/config`)
3. **System-wide config file** (`/etc/ssh/ssh_config` and `/etc/ssh/ssh_config.d/*.conf`)

---

## 4. SSH Client Configuration Files

### System-wide configuration:

```bash
cat /etc/ssh/ssh_config
```

### User configuration:

```bash
cat ~/.ssh/config
```

### Drop-in directory (system-wide):

```
/etc/ssh/ssh_config.d/*.conf
```

> 💡 Instead of modifying `/etc/ssh/ssh_config` directly, create drop-in files in `ssh_config.d/` (RHEL best practice).

### Example user config (`~/.ssh/config`):

```
Host serverb
    HostName serverb.datacenter.example.com
    User devops
    StrictHostKeyChecking yes
```

> 💡 With `StrictHostKeyChecking yes`, the `known_hosts` file **must** contain the host key — otherwise the connection fails.

---

## 5. Populating `known_hosts` — The Manual Way

### View existing `known_hosts`:

```bash
cat ~/.ssh/known_hosts
```

### Add a host key manually (not recommended):

```bash
ssh-keyscan serverb.datacenter.example.com >> ~/.ssh/known_hosts
```

### Add all hosts on a subnet (DANGEROUS):

```bash
ssh-keyscan 192.168.0.* >> ~/.ssh/known_hosts
```

---

## 6. The Danger of `ssh-keyscan`

| Problem | Impact |
|---------|--------|
| **No authenticity validation** | If someone intercepts the connection during the scan, a **fake key** could be recorded |
| **Defeats purpose of SSH security** | The whole point of SSH security is verifying the host key |

> ⚠️ `ssh-keyscan` is quick and convenient for pre-populating trusted keys, but **it does not validate the authenticity of each host**.

---

## 7. The Proper Way — Ansible

For managing SSH keys at scale, use **Ansible**.

### Ansible `known_hosts` module:

```yaml
- name: Add host key to known_hosts
  ansible.builtin.known_hosts:
    name: serverb.datacenter.example.com
    key: "{{ lookup('file', '/path/to/host-key.pub') }}"
    state: present
```

### Benefits of Ansible:

| Benefit | Description |
|---------|-------------|
| Centralized management | One playbook manages all systems |
| Validation | Keys can be verified before deployment |
| Automated removal | Outdated keys can be removed |
| Scale | Works for dozens, hundreds, or thousands of servers |

> 💡 Using Ansible, you can pre-populate the **system-wide** `known_hosts` file (`/etc/ssh/ssh_known_hosts`) for all users.

---

## 8. System-Wide `known_hosts`

### Location:

```
/etc/ssh/ssh_known_hosts
```

### Purpose:

- All users on the system benefit from the trusted keys
- No need to manage per-user `known_hosts` files

> 📌 This file may not exist by default — you create it.

---

## 9. Troubleshooting — Host Key Verification Failed

### Error message:

```
Host key verification failed.
```

### Causes:

| Cause | Solution |
|-------|----------|
| Host key not in `known_hosts` | Add the key |
| Host key changed (server reinstall) | Remove old key from `known_hosts`, add new one |
| `StrictHostKeyChecking yes` and key missing | Add key or change to `ask`/`accept-new` |

### Remove old key:

```bash
ssh-keygen -R serverb.datacenter.example.com
```

This removes the old key from `~/.ssh/known_hosts`.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View user known_hosts | `cat ~/.ssh/known_hosts` |
| View system known_hosts | `cat /etc/ssh/ssh_known_hosts` |
| Add host key (manual) | `ssh-keyscan serverb.datacenter.example.com >> ~/.ssh/known_hosts` |
| Scan entire subnet (dangerous) | `ssh-keyscan 192.168.0.* >> ~/.ssh/known_hosts` |
| Remove host key from known_hosts | `ssh-keygen -R serverb.datacenter.example.com` |
| View system SSH config | `cat /etc/ssh/ssh_config` |
| View user SSH config | `cat ~/.ssh/config` |
| View SSH client config man page | `man ssh_config` |
| View SSH server config man page | `man sshd_config` |

---

# SSH Client Configuration — Precedence

```
┌─────────────────────────────────────────────────────┐
│ 1. Command-line options (strongest)                  │
│    ssh -o StrictHostKeyChecking=yes serverb         │
├─────────────────────────────────────────────────────┤
│ 2. User config file (~/.ssh/config)                  │
│    Applies to the specific user only                 │
├─────────────────────────────────────────────────────┤
│ 3. System-wide config (/etc/ssh/ssh_config)          │
│    Applies to all users                              │
└─────────────────────────────────────────────────────┘
```

---

# `StrictHostKeyChecking` — Behavior Comparison

| Setting | New Host | Changed Host Key | Security Level |
|---------|----------|------------------|----------------|
| `ask` (default) | Prompts user | Prompts user | Balanced |
| `yes` | **Fails** | **Fails** | Maximum |
| `no` | Auto-accepts | Auto-accepts | **None** (dangerous) |
| `accept-new` | Auto-accepts | **Fails** | Good balance |

---

# Known Hosts File Format

```
# Format: hostname key-type key
servera.lab.example.com ecdsa-sha2-nistp256 AAAAB3NzaC1yc2EAAAADAQABAAABAQ...
serverb.lab.example.com ecdsa-sha2-nistp256 AAAAB3NzaC1yc2EAAAADAQABAAABAQ...
192.168.0.10 ecdsa-sha2-nistp256 AAAAB3NzaC1yc2EAAAADAQABAAABAQ...
```

- Each line = one host key
- Multiple aliases for same key are acceptable

---

# Key Takeaways

- **SSH host keys** = server fingerprints for identity verification.
- **`known_hosts`** file stores trusted host keys (user-level: `~/.ssh/known_hosts`, system-level: `/etc/ssh/ssh_known_hosts`).
- **`StrictHostKeyChecking`** controls how strict the client is about verifying host keys:
  - `ask` = default (prompt)
  - `yes` = maximum security (fails if key missing)
  - `no` = convenience (dangerous)
  - `accept-new` = good balance (new hosts accepted, changed keys rejected)
- **SSH client config** can be set system-wide or per-user:
  - `/etc/ssh/ssh_config` + `/etc/ssh/ssh_config.d/*.conf` (system)
  - `~/.ssh/config` (user)
- **`ssh-keyscan`** = quick but **dangerous** — it doesn't validate authenticity (vulnerable to man-in-the-middle attacks).
- **Proper way at scale:** Use **Ansible** with the `known_hosts` module — centrally manage keys, validate, and automate.
- **`ssh-keygen -R`** removes a host key from `known_hosts` (useful when a server has been rebuilt).