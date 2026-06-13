# Managing Applications with Flatpak — Notes

## 1. Core Concepts Recap

| Term | Meaning |
|------|---------|
| **Flatpak** | Application containerization for desktops |
| **Remote** | Flatpak repository (not called "repository" in Flatpak world) |
| **Application ID** | Unique identifier (e.g., `org.mozilla.firefox`) |
| **Sandbox** | Isolated environment for each application |

> 💡 Flatpak applications are **self-contained** — all dependencies bundled inside.

---

## 2. Authentication — The `auth.json` File

### When you run `podman login`:

```bash
podman login -u <username> -p <token> registry.redhat.io
```

### What gets created:

```bash
$XDG_RUNTIME_DIR/containers/auth.json
```

### Contents:

- Base64-encoded username + password/token
- Credentials for `registry.redhat.io` and `quay.io`

> ⚠️ **Security note:** `$XDG_RUNTIME_DIR` is under `/run/` — contents are **recreated on every boot**.

> ⚠️ **Never log in with real username/password.** Use **authentication tokens** (regenerate them regularly).

---

## 3. Copying Authentication for Flatpak

### For a single user (`~/.config/flatpak/`):

```bash
mkdir -p ~/.config/flatpak
cp $XDG_RUNTIME_DIR/containers/auth.json ~/.config/flatpak/oci-auth.json
```

### System-wide (all users — `/etc/flatpak/`):

```bash
sudo mkdir -p /etc/flatpak
sudo cp $XDG_RUNTIME_DIR/containers/auth.json /etc/flatpak/oci-auth.json
```

### Set proper permissions (system-wide):

```bash
sudo chmod 644 /etc/flatpak/oci-auth.json
```

### Permission breakdown (`644`):

| Digit | Who | Permission |
|-------|-----|------------|
| 6 | Owner (root) | Read + Write |
| 4 | Group | Read only |
| 4 | Others | Read only |

---

## 4. Remotes — Authentication Requirements

### Flathub (Fedora-based, free, no auth):

```bash
flatpak remote-ls flathub | wc -l
```

**Result:** ~5,329 desktop applications (no authentication required)

### RHEL remote (requires authentication):

```bash
flatpak remote-ls rhel
```

**Result:** ~8 applications (requires `registry.redhat.io` auth)

### Why the difference:

| Remote | Source | Authentication |
|--------|--------|----------------|
| `flathub` | Fedora community | None (free) |
| `rhel` | Red Hat | Required (customer only) |

---

## 5. Understanding Application IDs

### Format:

```
domain.organization.application-name
```

### Example:

```
org.mozilla.firefox
```

| Part | Meaning |
|------|---------|
| `org` | Domain (organization) |
| `mozilla` | Organization name |
| `firefox` | Application name |

### In `flatpak remote-ls` output:

| Column | Example | Meaning |
|--------|---------|---------|
| Name | `Firefox` | Simple name |
| Application ID | `org.mozilla.firefox` | Unique identifier |
| Version | `128.11.0` | Version number |
| Branch | `el10` | RHEL version branch |

---

## 6. Searching for Applications — `flatpak search`

```bash
flatpak search thunderbird
```

**Output example:**
```
Thunderbird      org.mozilla.thunderbird      ... rhel
Betterbird       eu.betterbird.Betterbird     ... flathub
Birdtray         com.ulduzsoft.Birdtray       ... flathub
Proton Mail      ...                          ... flathub
```

> 💡 Similar to `dnf search`, but for Flatpak applications.

---

## 7. Installing Applications — `flatpak install`

### Basic install:

```bash
flatpak install rhel org.mozilla.thunderbird
```

### Install without prompts (`-y`):

```bash
flatpak install -y rhel org.mozilla.thunderbird
```

### When multiple remotes provide the same app:

```bash
flatpak install -y thunderbird
```

**Prompt:**
```
Found similar refs:
 1) org.mozilla.thunderbird (flathub)
 2) org.mozilla.thunderbird (rhel)
Which do you want to use (0 to abort)? [0-2]: 2
```

> 💡 Choose the remote number (e.g., `2` for RHEL remote).

### Install with `--or-update`:

```bash
flatpak install -y --or-update eu.betterbird.Betterbird
```

| If application... | Action |
|-------------------|--------|
| Not installed | Installs it |
| Already installed | Updates to latest version |

---

## 8. Removing Applications — `flatpak remove`

```bash
flatpak remove -y thunderbird
```

### Options for removal:

| Option | Effect |
|--------|--------|
| (default) | Removes application only |
| `--delete-data` | Also removes user data |
| `--all` | Removes everything (app + data + dependencies) |

> 💡 Check `man flatpak-remove` for all options.

---

## 9. Updating Applications — `flatpak update`

### Update a specific application:

```bash
flatpak update eu.betterbird.Betterbird
```

### Update using Application ID (more explicit):

```bash
flatpak update org.mozilla.firefox
```

> 💡 Application ID is safer when multiple remotes offer similarly named apps.

---

## 10. Masking (Version Locking) — `flatpak mask`

### Prevent an application from being updated:

```bash
flatpak mask betterbird
```

### Check masked applications:

```bash
flatpak mask
```

### Remove the mask (allow updates again):

```bash
flatpak mask --remove betterbird
```

> 💡 Similar to `dnf versionlock` — prevents accidental updates.

---

## 11. Installing Specific Versions or Branches

### View available branches:

```bash
flatpak info com.redhat.Platform
```

**Output example:**
```
ID: com.redhat.Platform
Arch: x86_64
Branch: el10
Version: 10
```

### Install a specific branch:

```bash
flatpak install --noninteractive runtime/com.redhat.Platform/x86_64/el9
```

| Parameter | Meaning |
|-----------|---------|
| `--noninteractive` | No prompts, auto-yes |
| `runtime/` | Type (runtime vs application) |
| `com.redhat.Platform` | Application ID |
| `x86_64` | Architecture |
| `el9` | Branch (RHEL 9 version) |

> 💡 Runtimes are shared libraries — multiple applications can use the same runtime.

---

## 12. Flatpak Subcommands — Man Pages

### Main man page:

```bash
man flatpak
```

### Subcommand-specific man pages:

| Subcommand | Man Page |
|------------|----------|
| Install | `man flatpak-install` |
| Remove | `man flatpak-remove` |
| Update | `man flatpak-update` |
| Mask | `man flatpak-mask` |
| Search | `man flatpak-search` |
| Info | `man flatpak-info` |
| List | `man flatpak-list` |

> 💡 Use **tab completion** and **man pages** — don't memorize everything.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Search for applications | `flatpak search thunderbird` |
| Install from specific remote | `flatpak install rhel org.mozilla.thunderbird` |
| Install without prompts | `flatpak install -y rhel org.mozilla.thunderbird` |
| Install or update | `flatpak install -y --or-update eu.betterbird.Betterbird` |
| Remove application | `flatpak remove -y thunderbird` |
| Update application | `flatpak update eu.betterbird.Betterbird` |
| Mask (version lock) | `flatpak mask betterbird` |
| Remove mask | `flatpak mask --remove betterbird` |
| List masked apps | `flatpak mask` |
| Get app info | `flatpak info com.redhat.Platform` |
| List available apps in remote | `flatpak remote-ls rhel` |
| List configured remotes | `flatpak remotes` |
| Install specific branch | `flatpak install --noninteractive runtime/com.redhat.Platform/x86_64/el9` |
| Copy auth for user | `cp $XDG_RUNTIME_DIR/containers/auth.json ~/.config/flatpak/oci-auth.json` |
| Copy auth system-wide | `sudo cp $XDG_RUNTIME_DIR/containers/auth.json /etc/flatpak/oci-auth.json` |

---

# Authentication Flow for Flatpak

```
1. Generate token at access.redhat.com/terms-based-registry
              ↓
2. podman login -u <token-user> -p <token> registry.redhat.io
              ↓
3. Creates $XDG_RUNTIME_DIR/containers/auth.json
              ↓
4. Copy to ~/.config/flatpak/oci-auth.json (user) OR
   Copy to /etc/flatpak/oci-auth.json (system)
              ↓
5. Flatpak can now pull from rhel remote
```

---

# Remote Comparison

| Feature | Flathub | RHEL Remote |
|---------|---------|-------------|
| Source | Fedora community | Red Hat official |
| Authentication | None required | Required (registry.redhat.io) |
| Number of apps | ~5,300+ | ~8 |
| Support | Not supported by Red Hat | Fully supported |
| Use case | Additional desktop apps | Enterprise, supported apps |

---

# Key Takeaways

- **Remotes** = Flatpak repositories (e.g., `rhel`, `flathub`).
- **Authentication** is handled via `auth.json` → copy to `oci-auth.json` in Flatpak config directories.
- **`$XDG_RUNTIME_DIR`** is under `/run/` — contents lost on reboot (intentional for security).
- **Application ID format:** `domain.organization.app-name` (e.g., `org.mozilla.firefox`).
- **`flatpak search`** works like `dnf search` — finds applications across remotes.
- **`flatpak install -y --or-update`** = install if missing, update if present.
- **`flatpak mask`** = version locking (prevents updates).
- Use **Application IDs** when multiple remotes offer similarly named apps.
- Use **`--noninteractive`** for scripting (no prompts).
- **Man pages** (`man flatpak-<subcommand>`) are your best friend for learning.
- **Flatpak runtimes** (like `com.redhat.Platform`) are shared libraries — install specific branches with `runtime/` prefix.
- **Third-party remotes (Flathub) are not supported by Red Hat** — use at your own risk for development/testing.