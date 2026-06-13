# Using Flatpak — Notes

## 1. The Problem Flatpak Solves

### Traditional RPM dependency management:

```
Repository 1                    Repository 2
├── app1                        ├── app2
│   ├── white dep v1.1          │   ├── green dep v1.2
│   ├── purple dep v2.2         │   ├── purple dep v2.3 (conflict!)
│   └── green dep v3.3          │   └── white dep v1.1
```

### The conflict:

| Scenario | Result |
|----------|--------|
| Install `app1` | Works fine (purple dep v2.2) |
| Install `app2` | Updates purple dep to v2.3 |
| After update | `app1` may **break** (needs v2.2) |

> 💡 Traditional RPM = **system-wide** dependencies. One version per library.

---

## 2. The Flatpak Solution — Sandboxed Applications

### How Flatpak works:

```
Flatpak Repository 1            Flatpak Repository 2
├── app1 (sandboxed)            ├── app2 (sandboxed)
│   ├── white dep v1.1          │   ├── green dep v1.2
│   ├── purple dep v2.2         │   ├── purple dep v2.3
│   └── green dep v3.3          │   └── white dep v1.1
```

### Key benefits:

| Benefit | Description |
|---------|-------------|
| **Isolation** | Each app runs in its own sandbox |
| **No conflicts** | Different dependency versions can coexist |
| **Container-like** | Similar to containers, but for desktop apps |
| **System safe** | Cannot break other applications |

> 📌 Flatpak = **application containerization for desktops** (GUI applications).

---

## 3. Prerequisites — Flatpak Installation

### Required for Flatpak:

- Graphical user interface (GUI) — Flatpak is useless on headless servers
- `redhat-flatpak-repo` package (includes Red Hat's Flatpak repository)

### View files installed by the package:

```bash
rpm -ql redhat-flatpak-repo
```

**Key file:** `/etc/flatpak/remotes.d/rhel.flatpakrepo`

### View repository definition:

```bash
cat /etc/flatpak/remotes.d/rhel.flatpakrepo
```

> 💡 In Flatpak terminology, repositories are called **remotes**.

---

## 4. Viewing Available Flatpak Applications

### List applications in a remote:

```bash
flatpak remote-ls rhel
```

**Example output:**
```
org.mozilla.firefox
org.mozilla.thunderbird
...
```

### Compare versions — RPM vs Flatpak:

```bash
rpm -q firefox                    # RPM version (e.g., 128.10.0)
flatpak remote-ls rhel | grep firefox   # Flatpak version (e.g., 128.11.0)
```

> 💡 You can have **both versions installed simultaneously** without conflict.

---

## 5. Authentication — Registry Login

### Flatpak apps from Red Hat require authentication to `registry.redhat.io`

### Get credentials:

1. Go to: `access.redhat.com/terms-based-registry`
2. Generate an authentication token

### Login to the registry:

```bash
podman login -u <username> -p <token> registry.redhat.io
```

> ⚠️ Do not use your actual Red Hat password. Use the **token** as the password.

### What this does:

- Authenticates you to pull Flatpak applications
- Same registry used for container images

---

## 6. Installing Flatpak Applications

### Command syntax:

```bash
flatpak install <remote> <application-id>
```

### Example — install Firefox from rhel remote:

```bash
flatpak install rhel org.mozilla.firefox
```

### View installed Flatpak applications:

```bash
flatpak list
```

**Output example:**
```
org.mozilla.firefox              128.11.0  stable  rhel
io.github.josephmawa.TxtCompare  1.0.0     stable  flathub
```

---

## 7. Running Flatpak Applications

### Command syntax:

```bash
flatpak run <application-id>
```

### Run in background (detach from terminal):

```bash
flatpak run org.mozilla.firefox &
```

> 💌 The `&` detaches the process, returning control to the terminal.

### Run Text Compare example:

```bash
flatpak run io.github.josephmawa.TxtCompare &
```

---

## 8. Adding Third-Party Remotes (Not Supported by Red Hat)

### Add a remote:

```bash
flatpak remote-add flathub https://flathub.org/repo/flathub.flatpakrepo
```

### View all configured remotes:

```bash
flatpak remotes
```

**Output:**
```
rhel       system
flathub    system
```

### List applications from third-party remote:

```bash
flatpak remote-ls flathub
```

> ⚠️ **Warning:** Red Hat does **not support** third-party Flatpak repositories (e.g., Flathub).

---

## 9. Managing Flatpak via GUI

### Navigate to:

```
Red Hat logo (top left) → Menu → Software Repositories
```

### GUI capabilities:

| Feature | Description |
|---------|-------------|
| View remotes | See configured repositories |
| Enable/disable | Toggle repositories on/off |
| View installed apps | See all Flatpak applications |
| Install/remove | GUI-based package management |

> 💡 Both CLI and GUI methods are available for managing Flatpak.

---

## 10. Important Flatpak Commands

| Command | Purpose |
|---------|---------|
| `flatpak remote-ls <remote>` | List available apps in a remote |
| `flatpak install <remote> <app-id>` | Install an application |
| `flatpak list` | List installed applications |
| `flatpak run <app-id>` | Run an application |
| `flatpak remotes` | List configured remotes |
| `flatpak remote-add <name> <url>` | Add a new remote |
| `flatpak history` | Show transaction history |

### Get help:

```bash
man flatpak
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View Flatpak remote contents | `flatpak remote-ls rhel` |
| Install Flatpak app | `flatpak install rhel org.mozilla.firefox` |
| List installed Flatpak apps | `flatpak list` |
| Run Flatpak app | `flatpak run org.mozilla.firefox` |
| Run in background | `flatpak run org.mozilla.firefox &` |
| List configured remotes | `flatpak remotes` |
| Add new remote | `flatpak remote-add flathub <url>` |
| View remote repo file | `cat /etc/flatpak/remotes.d/rhel.flatpakrepo` |
| Login to registry | `podman login -u <user> -p <token> registry.redhat.io` |
| Get Flatpak help | `man flatpak` |

---

# RPM vs Flatpak — Comparison

| Feature | RPM | Flatpak |
|---------|-----|---------|
| Scope | System-wide | Per-application sandbox |
| Dependencies | Shared system-wide | Bundled per app |
| Version conflicts | Can occur | Isolated — no conflicts |
| Use case | All software | GUI desktop applications |
| Security | Standard Linux permissions | Sandboxed (restricted access) |
| Multiple versions | Cannot coexist (generally) | Can coexist freely |

---

# Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                     RHEL 10 Desktop                          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────┐    ┌─────────────────────┐         │
│  │   Flatpak Sandbox   │    │   Flatpak Sandbox   │         │
│  │   ┌─────────────┐   │    │   ┌─────────────┐   │         │
│  │   │  Firefox    │   │    │   │Text Compare │   │         │
│  │   │  v128.11.0  │   │    │   │  v1.0.0     │   │         │
│  │   └─────────────┘   │    │   └─────────────┘   │         │
│  │   Dependencies:     │    │   Dependencies:     │         │
│  │   • lib v2.2        │    │   • lib v2.3        │         │
│  │   • lib v1.1        │    │   • lib v1.2        │         │
│  └─────────────────────┘    └─────────────────────┘         │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              RPM-Installed Firefox                   │    │
│  │                   v128.10.0                          │    │
│  │         (system-wide dependencies)                   │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

---

# Key Takeaways

- **Flatpak** = application containerization for **desktop (GUI)** applications.
- Solves the **dependency conflict** problem — different apps can use different versions of the same library.
- Flatpak applications run **sandboxed** and **isolated** from the system.
- Red Hat provides an official Flatpak repository (remote) via `redhat-flatpak-repo`.
- Authentication to `registry.redhat.io` is required (use **token**, not password).
- **`flatpak remote-ls`** = list available apps. **`flatpak install`** = install. **`flatpak run`** = launch.
- Red Hat does **not support** third-party Flatpak repos like Flathub (but they work).
- Both **CLI** (`flatpak` command) and **GUI** (Software Repositories) management available.
- You can run **multiple versions** of the same application (e.g., Firefox via RPM and Flatpak side by side).
- Perfect for: GUI apps where you need the latest version without breaking system dependencies.