# Configuring SSH Key-Based Authentication — Notes

## 1. Why SSH Keys?

- **More secure** than password authentication
- Passwords **never leave** the client system
- Private key remains on the client

### How it works:

```
SSH Client (servera)          SSH Server (serverb)
       │
       │ 1. Generate key pair
       │    ssh-keygen -t ed25519
       │
       │ 2. Copy public key to server
       │    ssh-copy-id user@serverb
       │
       ▼
  Private key stays here
  (id_ed25519)
                              ~/.ssh/authorized_keys
                              (contains public key)
```

---

## 2. The SSH Key Pair

### Generate a key pair:

```bash
ssh-keygen -t ed25519
```

| Option | Meaning |
|--------|---------|
| `-t ed25519` | Algorithm (default for RHEL 10, more secure than RSA) |

### What gets created:

| File | Type | Purpose |
|------|------|---------|
| `~/.ssh/id_ed25519` | **Private key** | Keep secret! Never share |
| `~/.ssh/id_ed25519.pub` | **Public key** | Copy to servers (`authorized_keys`) |

### With passphrase (optional):

```bash
ssh-keygen -t ed25519
# Prompts: Enter passphrase (empty for no passphrase)
```

> 💡 A passphrase adds an extra layer of security — even if someone steals your private key, they need the passphrase to use it.

### Without passphrase (for automation):

```bash
ssh-keygen -t ed25519 -N ""
```

| Option | Meaning |
|--------|---------|
| `-N ""` | No passphrase (empty) |

### Multiple keys for same user:

- User can have **several public keys** in `~/.ssh/authorized_keys`
- Each key is stored on a separate line
- Useful when multiple clients connect as the same user

---

## 3. Copying the Public Key — `ssh-copy-id`

### Basic usage:

```bash
ssh-copy-id user@serverb
```

### With specific identity file:

```bash
ssh-copy-id -i ~/.ssh/id_student.pub user@serverb
```

### What `ssh-copy-id` does:

1. Connects to the server using password authentication
2. Appends the public key to `~/.ssh/authorized_keys`
3. Sets correct permissions on `~/.ssh/` and `authorized_keys`

> 💡 The public key is stored in `~/.ssh/authorized_keys` on the server.

---

## 4. Using SSH Agent — `ssh-agent` and `ssh-add`

### Problem:

- Private key has a passphrase
- You don't want to type it every time

### Solution — Use `ssh-agent`:

```bash
eval $(ssh-agent)
```

### Add your private key to the agent:

```bash
ssh-add ~/.ssh/id_student
```

### Automate in `~/.bashrc`:

```bash
eval $(ssh-agent) > /dev/null
ssh-add ~/.ssh/id_student
```

### Result:

- First use: prompt for passphrase once
- Subsequent uses: passphrase is cached

---

## 5. Client Configuration — `~/.ssh/config`

### Example configuration:

```
Host serverb
    HostName serverb.datacenter.example.com
    User devops
    StrictHostKeyChecking yes
    IdentityFile ~/.ssh/id_student
```

| Directive | Meaning |
|-----------|---------|
| `Host` | Alias for the connection |
| `HostName` | Actual hostname or IP |
| `User` | Username to log in as |
| `StrictHostKeyChecking` | Security level (yes/ask/no/accept-new) |
| `IdentityFile` | Specific private key to use |

### Connection without config:

```bash
ssh -i ~/.ssh/id_student devops@serverb.datacenter.example.com
```

### With config:

```bash
ssh serverb
```

> 💡 The config file simplifies connection and enforces security settings.

---

## 6. Testing — Forcing Password Authentication

### Force password authentication for testing:

```bash
ssh -o PreferredAuthentications=password student@serverb
```

### Or in config:

```
PreferredAuthentications=password
```

> 💡 Useful for testing when key authentication is not working.

---

## 7. Server-Side Configuration — `sshd_config`

### Configuration file location:

| Type | Location |
|------|----------|
| SSH **client** config | `/etc/ssh/ssh_config.d/*.conf` |
| SSH **server** config | `/etc/ssh/sshd_config.d/*.conf` |

> 💡 Use **drop-in directories** (`*_config.d/`) instead of modifying the main config files directly.

### Important server directives:

| Directive | Purpose | Default |
|-----------|---------|---------|
| `AllowGroups` | Only allow users in these groups | (none) |
| `AllowUsers` | Only allow specific users | (none) |
| `PasswordAuthentication` | Allow password login | `yes` |
| `PermitRootLogin` | Allow root login | `prohibit-password` |

### Example — `10-lab.conf`:

```
# /etc/ssh/sshd_config.d/10-lab.conf
AllowGroups wheel
PasswordAuthentication no
```

---

## 8. Reloading SSH Configuration

When you change the server config, **reload** (not restart) the service:

```bash
systemctl reload-or-restart sshd
```

| Action | Effect |
|--------|--------|
| `reload` | No downtime, sends SIGHUP |
| `restart` | Service interruption |
| `reload-or-restart` | Reload if possible, restart if not |

> 💡 `reload` is preferred — no disruption to existing connections.

---

## 9. Troubleshooting — `ssh -v`

### Increase verbosity to debug:

```bash
ssh -v user@serverb
ssh -vv user@serverb
ssh -vvv user@serverb
```

| Verbosity | Detail |
|-----------|--------|
| `-v` | Basic debug info |
| `-vv` | More detail |
| `-vvv` | Maximum detail (verbose mode) |

### What to look for:

```
debug1: Authentications that can continue: publickey,password
debug1: Offering public key: /home/student/.ssh/id_ed25519
debug1: Authentication succeeded (publickey)
```

> 💡 If you're stuck on an exam, use `man sshd_config` and search for the directive you need.

---

## 10. Testing with `sshpass` (Non-Interactive)

### Install:

```bash
dnf install -y sshpass
```

### Test password authentication (scripting):

```bash
sshpass -p redhat123 ssh -o PreferredAuthentications=password ricardo@serverb
```

> 💡 Great for automated testing, but **never hardcode passwords** in production.

---

## 11. User and Group Restrictions

### Allow only users in `wheel` group:

```
AllowGroups wheel
```

### Test:

```bash
# User not in wheel → access denied
ssh ricardo@serverb

# Add user to wheel group
usermod -aG wheel ricardo

# Now access is allowed
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Generate SSH key pair | `ssh-keygen -t ed25519` |
| Generate without passphrase | `ssh-keygen -t ed25519 -N ""` |
| Copy public key to server | `ssh-copy-id user@serverb` |
| Copy with specific identity | `ssh-copy-id -i ~/.ssh/id_student.pub user@serverb` |
| Start SSH agent | `eval $(ssh-agent)` |
| Add key to agent | `ssh-add ~/.ssh/id_student` |
| View client config | `cat ~/.ssh/config` |
| View server config | `cat /etc/ssh/sshd_config.d/*.conf` |
| Force password auth (client) | `ssh -o PreferredAuthentications=password user@serverb` |
| Force key auth (client) | `ssh -o PreferredAuthentications=publickey user@serverb` |
| Reload SSH server config | `systemctl reload-or-restart sshd` |
| Debug with verbosity | `ssh -vvv user@serverb` |
| View SSH server config man page | `man sshd_config` |
| View SSH client config man page | `man ssh_config` |

---

# SSH Key Location Summary

| Component | Location (Client) | Location (Server) |
|-----------|-------------------|-------------------|
| Private key | `~/.ssh/id_ed25519` | Never stored |
| Public key | `~/.ssh/id_ed25519.pub` | `~/.ssh/authorized_keys` |
| Client config | `~/.ssh/config` | N/A |
| Server config | N/A | `/etc/ssh/sshd_config.d/*.conf` |

---

# Common Server Directives

| Directive | Purpose | Use case |
|-----------|---------|----------|
| `AllowGroups wheel` | Only users in `wheel` can login | Administrative access |
| `PasswordAuthentication no` | Disable password login | Force key-based auth |
| `PermitRootLogin prohibit-password` | Root can use keys but not passwords | Security best practice |
| `AllowUsers student admin` | Only specific users can login | Restrict access |

---

# Key Takeaways

- **SSH keys** are more secure than passwords — passwords never leave the client.
- **`ssh-keygen -t ed25519`** generates the key pair (RHEL 10 default algorithm).
- **Private key** stays on client (`~/.ssh/id_ed25519`) — protect it.
- **Public key** goes to server (`~/.ssh/authorized_keys`) — can be copied with `ssh-copy-id`.
- **Passphrase** adds extra security; `ssh-agent` caches it so you don't type repeatedly.
- **`~/.ssh/config`** simplifies connections and enforces settings like `StrictHostKeyChecking`.
- **Server config** goes in `/etc/ssh/sshd_config.d/*.conf` — use `reload-or-restart` after changes.
- **AllowGroups/AllowUsers** restrict who can log in.
- **PasswordAuthentication no** forces key-based auth (more secure).
- **`ssh -vvv`** helps debug connection issues.
- **`man sshd_config`** and **`man ssh_config`** are your friends — especially on exams.