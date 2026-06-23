# Installing RHEL Using Image Mode — Notes

## 1. What Is Image Mode Installation?

- **Image mode** = OSTree-based edition of RHEL
- Boot from **bootc images** pulled from a container image registry
- The image is merged into the underlying RHEL system
- Requires a **qcow2** image (or ISO, cloud image, etc.)

```
Container Registry → bootc Image → OSTree Merge → RHEL System
```

---

## 2. Creating a qcow2 Image — `bootc-image-builder`

### The workflow:

1. Create a directory for the qcow2 file
2. Use `podman run` with `bootc-image-builder`
3. Generate a `.qcow2` file

### Command:

```bash
mkdir bootc-os
sudo podman run \
  --rm \
  --privileged \
  -v ./bootc-os:/output \
  -v ./config.toml:/config.toml \
  registry.redhat.io/rhel10/bootc-image-builder:latest \
  --type qcow2 \
  --config /config.toml \
  rhel-bootc:latest
```

| Option | Meaning |
|--------|---------|
| `--rm` | Remove container after execution |
| `--privileged` | Needed for disk/image operations |
| `-v ./bootc-os:/output` | Mount output directory |
| `-v ./config.toml:/config.toml` | Mount configuration file |
| `--type qcow2` | Output format |
| `--config` | Configuration file for user/SSH keys |
| `rhel-bootc:latest` | Base bootc image |

---

## 3. Configuration File — `config.toml`

### Example:

```toml
[[users]]
name = "rgdacosta"
password = "redhat123"
key = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC..."
groups = ["wheel"]
```

| Field | Meaning |
|-------|---------|
| `name` | Username |
| `password` | User password |
| `key` | SSH public key (for key-based authentication) |
| `groups` | Groups to add user to (`wheel` = sudo access) |

> 💡 Using a **configuration file** ensures that users and SSH keys are baked into the image.

---

## 4. Resulting qcow2 File

After the command completes:

```bash
ls -l bootc-os/
```

**Output:**
```
-rw-r--r-- 1 root root 2.5G disk.qcow2
```

> 💡 This qcow2 file can be used as a **virtual machine disk**.

---

## 5. Deploying the qcow2 — `virt-install`

### Create a new virtual machine:

```bash
virt-install \
  --name rhel10-bootc \
  --memory 4096 \
  --vcpus 2 \
  --disk /path/to/disk.qcow2 \
  --os-variant rhel10 \
  --network bridge=br0 \
  --noautoconsole
```

| Option | Meaning |
|--------|---------|
| `--name` | VM name |
| `--memory` | RAM in MB |
| `--vcpus` | Number of virtual CPUs |
| `--disk` | Path to qcow2 disk |
| `--os-variant` | OS type (rhel10) |
| `--network` | Network bridge |
| `--noautoconsole` | Don't open console automatically |

### Verify VM is running:

```bash
virsh list
```

---

## 6. Testing the Installed System

### Connect via SSH:

```bash
ssh rgdacosta@rhel10-bootc
```

### Check the OS version:

```bash
cat /etc/redhat-release
```

**Output:**
```
Red Hat Enterprise Linux 10.0 (Ostree)
```

> 💡 The `(Ostree)` suffix indicates this is an **image mode** installation.

---

## 7. Alternative — Kickstart Installation with bootc

Instead of using a qcow2 file, you can install with **Kickstart**:

```
ostreesetup --osname=rhel --remote=rhel-bootc --url=oci://registry.redhat.io/rhel-bootc:latest
```

### In the Kickstart file:

```
%pre
# Pull the bootc image
%end

%post
# Configure bootc
%end
```

> 💡 This approach allows you to **reboot into a bootc image** directly.

---

## 8. bootc Installation — Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Container Registry                       │
│             (registry.redhat.io / quay.io)                  │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│              bootc-image-builder                             │
│    (creates qcow2, ISO, or cloud image from bootc image)    │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                    Deployment Target                         │
│    • Bare metal (ISO)                                        │
│    • Virtual machine (qcow2)                                │
│    • Cloud instance (cloud image)                           │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│              OSTree Merge + Boot                             │
│    • Image is merged into the system                         │
│    • Reboot required                                         │
│    • Application runs consistently                           │
└─────────────────────────────────────────────────────────────┘
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Create qcow2 image | `podman run --rm --privileged -v ./output:/output -v ./config.toml:/config.toml registry.redhat.io/rhel10/bootc-image-builder:latest --type qcow2 --config /config.toml rhel-bootc:latest` |
| Create VM from qcow2 | `virt-install --name rhel10-bootc --memory 4096 --vcpus 2 --disk /path/to/disk.qcow2 --os-variant rhel10 --network bridge=br0 --noautoconsole` |
| List running VMs | `virsh list` |
| Connect to VM | `ssh user@vm-ip` |
| Check OS version | `cat /etc/redhat-release` |
| View bootc images | `podman images` |
| Push bootc image | `podman push image:tag` |

---

# Image Mode Installation — Overview

| Component | Purpose |
|-----------|---------|
| **bootc image** | Complete OS image (kernel + systemd + applications) |
| **bootc-image-builder** | Tool to create deployable artifacts (qcow2, ISO) |
| **config.toml** | User, SSH key, and group configuration |
| **qcow2** | Virtual machine disk image |
| **OSTree** | Technology that merges the image into the system |

---

# Key Takeaways

- **Image mode** = OSTree-based RHEL, booting from container images.
- **`bootc-image-builder`** creates qcow2, ISO, or cloud images from bootc images.
- **Configuration file (`config.toml`)** defines users, passwords, SSH keys, and groups.
- **qcow2 images** are used for virtual machine deployments.
- **`virt-install`** creates a VM from the qcow2 image.
- **OSTree suffix** (`/etc/redhat-release` shows `(Ostree)`) indicates image mode.
- **Kickstart** can also be used to deploy bootc images.
- **Updates** = build new image → push to registry → reboot (just like application containers).
- **Rollback** = single command + reboot.
- **Consistency, security, and reliability** are the key benefits.