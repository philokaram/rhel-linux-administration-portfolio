# Managing Image Mode–Based Systems — Notes

## 1. The `bootc` Command — Status

### View current boot status:

```bash
bootc status
```

**Output shows:**
- **Booted image** — currently running
- **Staged image** — downloaded, awaiting reboot
- **Rollback image** — previous image (for rollback)

### Example output:

```
Booted image: quay.io/rdacosta/bootc_httpd:v1 (digest: abc123...)
Staged image: quay.io/rdacosta/bootc_httpd:v2.1 (digest: def456...)
Rollback image: quay.io/rdacosta/bootc_httpd:v2 (digest: 789ghi...)
```

> 💡 The **digest** uniquely identifies the image — even with the same tag, the digest changes.

---

## 2. Switching Images — `bootc switch`

### Switch to a specific image:

```bash
bootc switch quay.io/rdacosta/bootc_httpd:v2.1
```

**What happens:**
1. Downloads the new image
2. Staged image is updated
3. `bootc status` shows `Staged` image
4. **Reboot required** to apply

### Verify:

```bash
bootc status
```

---

## 3. Automatic Updates — Systemd Timer

### Timer unit:

```bash
bootc-fetch-apply-updates.timer
```

### What it does:

- Periodically checks the image registry
- If a new image is available, it's downloaded
- You control **when to reboot**

### Disable automatic updates:

```bash
systemctl mask bootc-fetch-apply-updates.timer
```

### Enable automatic updates:

```bash
systemctl unmask bootc-fetch-apply-updates.timer
```

> 💡 **Masking** prevents the timer from running at all.

---

## 4. Manual Updates — `bootc upgrade` / `bootc update`

### Check for updates (without pulling):

```bash
bootc upgrade
```

### Download the latest version of the current image:

```bash
bootc upgrade
```

### Pull a specific image version:

```bash
bootc switch quay.io/rdacosta/bootc_httpd:v2.1
```

> 💡 `bootc upgrade` and `bootc update` are functionally the same.

---

## 5. Rollback — `bootc rollback`

### Rollback to the previous image:

```bash
bootc rollback
```

### What happens:

1. Staged image is set to the previous image
2. A **reboot** is required
3. The system boots into the rollback image

### Verify:

```bash
bootc status
```

**Output:**
```
Booted image: quay.io/rdacosta/bootc_httpd:v1
Rollback image: quay.io/rdacosta/bootc_httpd:v2.1
```

---

## 6. Image Versioning — Tags vs Digests

| Concept | Example | Meaning |
|---------|---------|---------|
| **Tag** | `v1`, `v2.1`, `latest` | Human-readable label (can change) |
| **Digest** | `sha256:abc123...` | Immutable identifier (always unique) |

### Why tags can be confusing:

- `v1` tag may point to **different** images over time (floating tag)
- The **digest** is the true identifier

> 💡 When rolling back, the **digest** ensures the exact image is used.

---

## 7. Bootc Workflow with Git

### Structure:

```
Git Repository
├── main
│   └── Containerfile
├── version1
│   └── Containerfile
└── version2
    └── Containerfile
```

### Workflow:

1. Make changes to `Containerfile`
2. Build the new image:

```bash
podman build -t bootc_httpd:v2.1 -f Containerfile .
```

3. Push to registry:

```bash
podman push quay.io/rdacosta/bootc_httpd:v2.1
```

4. On the bootc host:

```bash
bootc switch quay.io/rdacosta/bootc_httpd:v2.1
reboot
```

5. Test the new version
6. If issues → rollback:

```bash
bootc rollback
reboot
```

---

## 8. Bootc Command Summary

| Command | Purpose |
|---------|---------|
| `bootc status` | Show current, staged, and rollback images |
| `bootc switch <image>` | Switch to a specific image |
| `bootc upgrade` / `update` | Pull the latest version of the current image |
| `bootc rollback` | Rollback to the previous image |
| `bootc --help` | Show all available subcommands |

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View boot status | `bootc status` |
| Switch to new image | `bootc switch quay.io/user/image:tag` |
| Pull latest updates | `bootc upgrade` |
| Rollback to previous | `bootc rollback` |
| Disable auto-updates | `systemctl mask bootc-fetch-apply-updates.timer` |
| Enable auto-updates | `systemctl unmask bootc-fetch-apply-updates.timer` |
| Reboot | `reboot` |
| View bootc help | `bootc --help` |
| View remote Git URL | `git remote get-url origin` |

---

# Bootc Image Lifecycle

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Containerfile (in Git)                                   │
│    └── Version controlled                                    │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Build image (podman build)                               │
│    └── Tag: v2.1                                            │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Push to registry (podman push)                           │
│    └── quay.io/user/bootc_httpd:v2.1                        │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Bootc host pulls image (bootc switch)                    │
│    └── Staged image updated                                  │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. Reboot                                                   │
│    └── Booted image = v2.1                                  │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 6. Test                                                    │
│    ├── Works → continue                                    │
│    └── Fails → bootc rollback → reboot                     │
└─────────────────────────────────────────────────────────────┘
```

---

# Key Takeaways

- **`bootc status`** shows current, staged, and rollback images.
- **`bootc switch <image:tag>`** switches to a new image (requires reboot).
- **`bootc upgrade`** pulls the latest version of the current image.
- **`bootc rollback`** reverts to the previous image (requires reboot).
- **Automatic updates** are controlled by `bootc-fetch-apply-updates.timer`.
- **Mask the timer** to disable automatic updates.
- **Tags** (e.g., `v1`, `v2.1`) are human-readable but can change.
- **Digests** are immutable and uniquely identify an image.
- **Git** is used for version control of Containerfiles.
- **Rollback** is instant — just a command + reboot.
- **Image mode** provides consistency, security, and reliability.