# Bootc — Image Mode for RHEL — Notes

## 1. The Problem — Traditional Server Management

### Traditional approach:

- Install RHEL
- Layer on applications, updates, configuration
- Each server ends up **slightly different**
- Patching, troubleshooting, and uncertainty over time

### The solution — **bootc**:

- A single **complete image** (blueprint for the whole system)
- Build once → deploy consistently everywhere
- **Immutable root file system** → secure and stable

> 💡 **Analogy:** Instead of building a server piece by piece, you create a full blueprint and deploy it exactly the same way every time.

---

## 2. What Is bootc?

- Brand new way to run Linux in RHEL 10
- Built from a **Containerfile** (like normal application images)
- Base image uses **RHEL** with **systemd** and all other components
- Images are stored in an **image registry**

### Key difference:

| Traditional RHEL | Image Mode (bootc) |
|------------------|-------------------|
| Built with RPMs | Built with a Containerfile |
| Servers drift over time | **Identical** images every time |
| Manual updates | Build new image → reboot |
| Patches applied separately | Everything updated at once |

---

## 3. bootc Workflow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Build bootc image from Containerfile                    │
│    └── Base: RHEL 10 + systemd + applications + config     │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Push image to image registry                            │
│    └── registry.redhat.io / quay.io / custom registry      │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Deploy to:                                              │
│    • Bare metal                                            │
│    • Virtual machine (qcow2)                               │
│    • Cloud instance                                        │
│    • ISO                                                   │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Manage fleet                                            │
│    • Publish new images to registry                        │
│    • Hosts automatically pull new images                   │
│    • Reboot to apply updates                               │
│    • Rollback with a single command                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 4. Immutable Root File System

### What is immutable?

- Root file system **cannot be changed** (read-only)
- Only **writable directories**:

| Directory | Purpose |
|-----------|---------|
| `/etc` | Configuration files |
| `/var` | Variable data |

### Benefits:

| Benefit | Description |
|---------|-------------|
| **Security** | Can't be compromised by unauthorized changes |
| **Stability** | No configuration drift |
| **Consistency** | Every server is identical |
| **Reliability** | Updates are atomic (all or nothing) |

---

## 5. Updates and Rollbacks

### Updating with bootc:

```
Old Image (v1.0) → Build New Image (v1.1) → Push to Registry
                                                    │
                                                    ▼
                                            Host pulls new image
                                                    │
                                                    ▼
                                            Reboot (fast, reliable)
                                                    │
                                                    ▼
                                            Running v1.1
```

### Rollback:

```
Host running v1.1 → Issue rollback command → Reboot
                                                    │
                                                    ▼
                                            Running v1.0 again
```

> 💡 **Rollback is just a single command** — fast and reliable.

---

## 6. Deployment Options

| Format | Use Case |
|--------|----------|
| **qcow2** | Virtual machines (KVM, VMware) |
| **ISO** | Physical servers (bare metal) |
| **Cloud image** | AWS, Azure, GCP |

### Kickstart integration:

- You can use **Kickstart** to install a server with a bootc base

---

## 7. Automatic Pulling — systemd Timer

- Hosts can automatically pull new images from the registry
- Controlled by a **systemd timer unit**
- You decide **when to reboot** (after pulling)
- Can be **disabled** if you want manual control

### Version control:

- Images are tagged (e.g., `v1.0`, `v1.1`)
- Tags facilitate version control and rollbacks

---

## 8. Key Benefits Summary

| Benefit | Description |
|---------|-------------|
| **Consistency** | Every server is identical |
| **Simplicity** | Build once → deploy everywhere |
| **Security** | Immutable root, minimal attack surface |
| **Reliability** | Atomic updates, easy rollback |
| **Speed** | Fast reboots instead of complex upgrade processes |
| **Scalability** | Manage entire fleets from a single image |
| **Modern** | Container-based workflow |

---

# bootc — Command Reference

| Task | Command |
|------|---------|
| Build bootc image | `podman build -t my-server:latest -f Containerfile .` |
| Push to registry | `podman push my-server:latest` |
| Pull on host | (automatic via systemd timer) |
| Check current bootc image | (specific command TBD) |
| Rollback | (specific command TBD) |
| Disable auto-updates | `systemctl disable bootc-auto-update.timer` |

---

# Traditional vs bootc — Comparison

| Feature | Traditional RHEL | Image Mode (bootc) |
|---------|------------------|-------------------|
| Installation | RPM packages + config | Single container image |
| Configuration drift | ✅ Yes (over time) | ❌ No (identical images) |
| Updates | Package-by-package | Entire image at once |
| Rollback | Complex (downgrade packages) | Simple (rollback image) |
| Root file system | Writable | Immutable |
| Consistency | Low | High |
| Security | Moderate | High |
| Deployment speed | Slow (packages + config) | Fast (image pull + reboot) |

---

# Key Takeaways

- **bootc** = Image Mode for RHEL (brand new in RHEL 10).
- Build a **single complete image** from a Containerfile.
- Base image = RHEL + systemd + all components.
- **Immutable root file system** (only `/etc` and `/var` are writable).
- Updates = build new image → push to registry → reboot.
- Rollback = single command → reboot.
- **No configuration drift** — every server is identical.
- Works for: bare metal, virtual machines, cloud, ISO.
- **Kickstart** can be used to install with a bootc base.
- Auto-updates controlled by **systemd timer** (can be disabled).
- **bootc is fast, reliable, consistent, and secure.**