# SELinux Port Labeling — Notes

## 1. SELinux and Ports

- SELinux enforces rules on **ports** as well as files and processes
- Every port has an associated **SELinux type context**
- If a service tries to listen on a port that doesn't match its type, SELinux blocks it

### Policy categories (revisited):

| Category | Managed By |
|----------|------------|
| Files | `semanage fcontext` |
| Processes/Domains | `semanage permissive` |
| Users | `semanage user` |
| **Booleans** | `setsebool` |
| **Ports** | `semanage port` |

---

## 2. The Problem — Changing SSH Port

### Default SSH port:

```
/etc/ssh/sshd_config: Port 22
```

### Change to port 23:

```bash
vi /etc/ssh/sshd_config.d/90-custom.conf
```

**Content:**
```
Port 23
```

### Restart SSH:

```bash
systemctl restart sshd
```

**Result:** ❌ **FAILS** — SELinux blocks SSH from binding to port 23.

---

## 3. Viewing SELinux Port Mappings — `semanage port -l`

### List all port mappings:

```bash
semanage port -l
```

### Filter for SSH-related ports:

```bash
semanage port -l | grep ssh
```

**Output:**
```
ssh_port_t                     tcp      22
```

### Filter for specific ports:

```bash
semanage port -l | grep -E "\<(22|23)\>"
```

**Output:**
```
ssh_port_t                     tcp      22
telnetd_port_t                 tcp      23
```

> 💡 Port 23 is labeled `telnetd_port_t` — SSH (which expects `ssh_port_t`) cannot bind to it.

---

## 4. Adding a Port to an SELinux Type — `semanage port -a`

### Associate port 23 with `ssh_port_t`:

```bash
semanage port -a -t ssh_port_t -p tcp 23
```

| Option | Meaning |
|--------|---------|
| `-a` | **A**dd |
| `-t` | **T**ype context |
| `-p` | **P**rotocol (tcp/udp) |
| `23` | Port number |

### Verify:

```bash
semanage port -l | grep ssh
```

**Output:**
```
ssh_port_t                     tcp      23, 22
```

---

## 5. Modifying an Existing Port Mapping — `semanage port -m`

### If the port already has a different mapping:

```bash
semanage port -m -t ssh_port_t -p tcp 23
```

| Option | Meaning |
|--------|---------|
| `-m` | **M**odify |

> 💡 Use `-m` instead of `-a` if the port already exists with a different type.

---

## 6. Viewing Only Custom Port Mappings — `semanage port -l -C`

### List customizations only:

```bash
semanage port -l -C
```

**Output:**
```
ssh_port_t                     tcp      23
```

---

## 7. Web Console — SELinux Port Management

### Steps:

1. Connect to `https://server:9090`
2. Log in (e.g., `student`)
3. Turn on administrative access
4. Navigate to **Tools → SELinux**
5. Expand the violation
6. View the **solution** (may suggest adding the port)
7. Apply the solution from the web console

> 💡 **Work smarter:** Use the web console for visual, guided SELinux troubleshooting.

---

## 8. SELinux Policy Documentation

### Install policy documentation:

```bash
dnf install -y selinux-policy-doc
```

### View SELinux man pages for a specific service:

```bash
man -k _selinux | grep ssh
```

**Output:**
```
sshd_selinux (8)  - Security Enhanced Linux Policy for the sshd daemon
```

### View detailed SELinux policy docs:

```bash
man sshd_selinux
```

**Contents:**
- Booleans
- Port types
- File contexts
- Detailed explanations

---

## 9. Troubleshooting Workflow for SELinux Port Issues

```
1. Service fails to start
       │
       ▼
2. Check SELinux mode: getenforce
       │
       ▼
3. Check SELinux violations:
   - Web Console → Tools → SELinux
   - Or: journalctl -u service-name
       │
       ▼
4. Identify the port and expected type:
   semanage port -l | grep service_name
       │
       ▼
5. Add the port to the correct type:
   semanage port -a -t correct_type -p tcp PORT
       │
       ▼
6. Restart service: systemctl restart service-name
       │
       ▼
7. Verify: systemctl status service-name
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| List all port mappings | `semanage port -l` |
| List port mappings (filter) | `semanage port -l \| grep ssh` |
| List customizations only | `semanage port -l -C` |
| Add port to SELinux type | `semanage port -a -t ssh_port_t -p tcp 23` |
| Modify existing port mapping | `semanage port -m -t ssh_port_t -p tcp 23` |
| Delete port mapping | `semanage port -d -p tcp 23` |
| Install SELinux docs | `dnf install -y selinux-policy-doc` |
| Search SELinux man pages | `man -k _selinux \| grep service` |
| View service SELinux docs | `man sshd_selinux` |
| Check SSH status | `systemctl status sshd` |

---

# Common SELinux Port Types

| Type | Associated Ports |
|------|------------------|
| `ssh_port_t` | 22 |
| `http_port_t` | 80, 443, 8008, 8080 |
| `https_port_t` | 443 |
| `telnetd_port_t` | 23 |
| `smtp_port_t` | 25, 465, 587 |
| `dns_port_t` | 53 |
| `ntp_port_t` | 123 |
| `mysql_port_t` | 3306 |

---

# SELinux Port Labeling — Visual Flow

```
┌─────────────────────────────────────────────────────────────┐
│ Service tries to bind to port 23                            │
│ (SSH expects ssh_port_t)                                    │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
        ┌─────────────────────────┐
        │   semanage port -l      │
        │   grep 23               │
        └────────────┬────────────┘
                     │
                     ▼
        ┌─────────────────────────┐
        │   Port 23 is labeled    │
        │   telnetd_port_t        │
        └────────────┬────────────┘
                     │
                     ▼
        ┌─────────────────────────┐
        │   Mismatch!              │
        │   SELinux blocks SSH    │
        │   from binding          │
        └────────────┬────────────┘
                     │
                     ▼
        ┌─────────────────────────┐
        │   Fix:                   │
        │   semanage port -a      │
        │   -t ssh_port_t         │
        │   -p tcp 23             │
        └─────────────────────────┘
```

---

# Key Takeaways

- **SELinux labels ports** — every port has a type context.
- **Services can only bind** to ports with their expected SELinux type.
- **SSH expects** `ssh_port_t` (port 22 by default).
- **Port 23** is labeled `telnetd_port_t` (from legacy Telnet).
- **`semanage port -l`** shows all port-to-type mappings.
- **`semanage port -a`** adds a new port to a type.
- **`semanage port -m`** modifies an existing mapping.
- **`semanage port -l -C`** shows only custom (modified) mappings.
- **Web Console** provides a visual interface for SELinux port issues.
- **`selinux-policy-doc`** installs detailed SELinux man pages.
- **`man sshd_selinux`** shows all SELinux settings for SSH (booleans, ports, file contexts).