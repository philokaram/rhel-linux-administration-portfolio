# Installing RHEL 10 — Notes

## 1. Installation Methods

| Method | Description | Use Case |
|--------|-------------|----------|
| **Boot ISO** | Small ISO, downloads packages from CDN | Internet-connected systems |
| **Offline ISO** | ~10 GB, contains all packages | No internet access, local network |
| **Cloud image** | Pre-built for cloud providers | AWS, Azure, GCP deployments |
| **PXE** | Network boot with DHCP + web server | Large-scale, automated deployments |
| **RDP** | Remote display protocol | Headless servers |
| **Custom ISO** | Pre-configured with custom packages | Standardized deployments |

---

## 2. Getting RHEL

### Sources:

| Source | URL | Purpose |
|--------|-----|---------|
| Customer Portal | `access.redhat.com` | RHEL subscriptions |
| Developer Subscription | `developers.redhat.com` | **Free** for learning/testing (up to 16 systems) |

> 💡 Developer subscription gives full access to CDN content but **no support tickets** (knowledge base still available).

### Installation images:

- **Boot ISO**: Small (network install)
- **Offline ISO**: ~10 GB (complete)

---

## 3. Developer Subscription

- Free for learning and testing
- Register up to **16** RHEL systems
- Full access to:
  - Red Hat Content Delivery Network (CDN)
  - Knowledge Base articles
- **No** support tickets

---

## 4. Anaconda — Installation Program

- Anaconda is the RHEL installer
- Supports **interactive** and **non-interactive** (kickstart) installations

### Anaconda Screens:

| Section | Purpose |
|---------|---------|
| **LOCALIZATION** | Language, timezone, keyboard |
| **SOFTWARE** | Installation source, software selection |
| **SYSTEM** | Installation destination, network, hostname, security policy |
| **USER SETTINGS** | Root password, user creation |

---

## 5. RDP Installation — Headless Servers

### What is RDP installation?

- Install RHEL on a **headless server** (no display)
- Connect remotely using an RDP client

### Boot options required:

```
inst.rdp
```

### Additional options:

```
rdp.password=<password>
```

### Connect:

```
VNC client to server:5900
```

---

## 6. PXE Installation — Network Boot

### Components:

| Component | Purpose |
|-----------|---------|
| **DHCP** | Supplies IP address |
| **PXE** | Turns network card into DHCP client (no OS) |
| **Web server** | Hosts the installation ISO |

### Boot menu:

- Customizable via PXE server
- Options include:
  - Normal installation
  - RDP installation

---

## 7. Anaconda — Key Configuration Settings

### Default changes in RHEL 10:

| Setting | RHEL 10 Default |
|---------|-----------------|
| **Root account** | **Disabled** by default |
| **User creation** | Must create a user account |
| **Admin user** | User can be made member of `wheel` group (sudo access) |

### Installation destination:

- **Automatic partitioning** (default)
- Custom partitioning available
- LUKS encryption optional

### Kdump:

- **Enabled** by default
- Saves kernel crash dumps

### Software selection:

- Server with GUI (graphical interface)
- Minimal installation also available

---

## 8. Anaconda — Troubleshooting with Tmux

### Accessing TTYs:

| TTY | Purpose |
|-----|---------|
| **TTY1** (`Ctrl+Alt+F1`) | Tmux logs (console) |
| **TTY6** (`Ctrl+Alt+F6`) | Graphical Anaconda interface |

### Tmux navigation:

| Key | Action |
|-----|--------|
| `Ctrl+B` | Prefix (start Tmux command) |
| `Ctrl+B + 3` | Switch to log pane |
| `Ctrl+B + n` | Next pane |

### Tmux logs include:

- Installation logs (Anaconda)
- Program logs
- Storage logs

> 💡 Use logs to troubleshoot installation issues.

---

## 9. Post-Installation

1. **Reboot** the system after installation
2. **Log in** with the user account created during installation
3. **Use sudo** for administrative tasks

### Root access:

```bash
sudo -i
```

---

# Key Installation Options — Summary

| Option | Description |
|--------|-------------|
| `inst.rdp` | Enable RDP installation |
| `rdp.password=...` | Set RDP password |
| `inst.ks=...` | Kickstart file location |
| `inst.repo=...` | Installation repository URL |

---

# Installation Workflow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Obtain RHEL image (Boot ISO or Offline ISO)             │
│    - access.redhat.com or developers.redhat.com            │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Boot from installation media                            │
│    - Physical: DVD/USB                                      │
│    - Virtual: ISO attached                                  │
│    - Network: PXE boot                                      │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Anaconda installer                                       │
│    - Language → Timezone → Keyboard                         │
│    - Installation source (CDN or local)                    │
│    - Software selection                                     │
│    - Partitioning (automatic or manual)                    │
│    - User creation (root disabled, create user)            │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Installation process                                     │
│    - Monitor via Tmux logs (TTY1)                          │
│    - Switch to GUI (TTY6)                                  │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. First boot                                              │
│    - Log in with created user                              │
│    - Use sudo for admin tasks                              │
└─────────────────────────────────────────────────────────────┘
```

---

# TTY Summary

| TTY | Purpose |
|-----|---------|
| `Ctrl+Alt+F1` | Tmux (logs) |
| `Ctrl+Alt+F6` | Graphical Anaconda GUI |

---

# Key Takeaways

- **Anaconda** is the RHEL installer (interactive and non-interactive).
- **Boot ISO** = small, downloads packages from CDN (internet required).
- **Offline ISO** = ~10 GB, complete packages (no internet required).
- **Developer Subscription** = free for up to 16 systems (learning/testing).
- **Root account is disabled** by default in RHEL 10 (create a user with sudo).
- **RDP installation** = install on headless servers (connect with VNC client).
- **PXE installation** = network boot with DHCP + web server (large-scale deployments).
- **Anaconda GUI** is on TTY6 (`Ctrl+Alt+F6`).
- **Tmux logs** are on TTY1 (`Ctrl+Alt+F1`) — helpful for troubleshooting.
- **Developer subscription** = free RHEL for learning.