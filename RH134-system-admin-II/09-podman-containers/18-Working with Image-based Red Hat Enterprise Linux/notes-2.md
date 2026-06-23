# Creating Installable Images with bootc — Notes

## 1. The Problem — Configuration Drift

### Traditional system management problems:

- Every server develops small differences over time
- **Configuration drift** = troubleshooting difficulty + security compliance challenges

### Image mode solution:

- Define the operating system **as code** (Containerfile)
- Build a **single image**
- Deploy consistently everywhere
- **No drift** — every system is identical

---

## 2. bootc — Key Difference from Application Containers

| Feature | Application Container | bootc (OS Image) |
|---------|----------------------|------------------|
| Base image | Application runtime (UBI) | **RHEL + kernel + bootloader + systemd** |
| Purpose | Run an application | Run an entire operating system |
| Init system | Optional | **systemd** required |
| Deployment | Container runtime (podman) | Boot directly (bare metal/VM/cloud) |
| ENTRYPOINT | Yes (application command) | No (systemd starts everything) |

> 💡 **bootc images are full operating systems** — they boot, not just run in a container.

---

## 3. Building the bootc Image

### Containerfile example:

```dockerfile
# Layer 1: RHEL 10 base with bootloader, kernel, and systemd
FROM registry.redhat.io/rhel10/bootc:latest

# Add repository configuration
COPY classroom.repo /etc/yum.repos.d/

# Install applications and utilities
RUN dnf install -y httpd vim-enhanced && \
    dnf clean all

# Enable services (systemd)
RUN systemctl enable httpd

# Move application data to /usr/share (immutable)
RUN mv /var/www /usr/share/www && \
    sed -i 's|/var/www|/usr/share/www|g' /etc/httpd/conf/httpd.conf

# Copy application files
COPY index.html /usr/share/www/html/
COPY redhat-logo.jpeg /usr/share/www/html/

# Metadata only (does NOT open ports)
EXPOSE 80
```

> ⚠️ **No ENTRYPOINT** — systemd is the init process.

---

## 4. Key Directories — Immutable vs Writable

| Directory | Immutable? | Purpose |
|-----------|------------|---------|
| `/usr` | ✅ **Immutable** | System files, applications (read-only) |
| `/usr/share/www` | ✅ Immutable | Application data (moved from `/var/www`) |
| `/etc` | ❌ Writable | Configuration files |
| `/var` | ❌ Writable | Variable data (logs, spool, etc.) |

> 💡 Move application data to `/usr/share/` so it becomes immutable and consistent across deployments.

---

## 5. Building the Image — `podman build --squash`

### Command:

```bash
podman build --squash -t quay.io/rdacosta/httpd:v1.1 -f Containerfile.bootc
```

| Option | Meaning |
|--------|---------|
| `--squash` | Merge all layers into a single layer (efficient) |
| `-t` | Tag (image name + version) |
| `-f` | Specify the Containerfile (explicit) |

> 💡 **Why `-f`?** Avoids missing the `.` (build context) when using the dot approach.

---

## 6. Testing the Image Locally

### Run as a container for testing:

```bash
podman run -d -p 8081:80 quay.io/rdacosta/httpd:v1.1
```

### Verify:

```bash
curl http://localhost:8081
```

### Check inside the container:

```bash
podman exec -it <container_id> bash
```

**Commands inside:**

```bash
systemctl status httpd          # systemd is running!
ps -ef                          # init (systemd) is PID 1
ls /usr/share/www/html/         # Application files in immutable location
```

> 💡 Unlike traditional application containers, bootc containers have **systemd as PID 1**.

---

## 7. Firewall Configuration for Testing

### Add port to firewall (temporary + permanent):

```bash
firewall-cmd --add-port=8081/tcp
firewall-cmd --runtime-to-permanent
```

> 💡 Always test locally before pushing to the registry.

---

## 8. Pushing the Image to a Registry

```bash
podman push quay.io/rdacosta/httpd:v1.1
```

### After publishing:

- Images are available for installation
- All systems running image mode can pull updates
- **Version control** = tag management

---

## 9. bootc Workflow — CI/CD Integration

```
Git Repository (Containerfile)
         │
         ▼
CI/CD Pipeline (build + test)
         │
         ▼
Image Registry (published image)
         │
         ▼
Systems Pull and Reboot
         │
         ▼
Consistent Deployments
```

### Benefits:

| Benefit | Description |
|---------|-------------|
| **Reliability** | Only validated images are published |
| **Consistency** | Every system pulls from the same source |
| **Compliance** | Simplified auditing |
| **Rollback** | Instant rollback to previous image |

---

## 10. Rollback — Instant Recovery

```
System running v1.1
         │
         ▼
Issue rollback command
         │
         ▼
Reboot
         │
         ▼
Running v1.0 again
```

> 💡 **No complex recovery steps** — just a single command and a reboot.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Build bootc image | `podman build --squash -t image:tag -f Containerfile.bootc .` |
| Run image for testing | `podman run -d -p 8081:80 image:tag` |
| Check running containers | `podman ps` |
| Execute inside container | `podman exec -it container_id bash` |
| Add firewall port | `firewall-cmd --add-port=8081/tcp` |
| Make firewall permanent | `firewall-cmd --runtime-to-permanent` |
| Push image to registry | `podman push image:tag` |
| View image metadata | `podman image inspect image:tag` |

---

# Application Container vs bootc OS Image — Comparison

| Feature | Application Container | bootc OS Image |
|---------|----------------------|----------------|
| Base image | UBI (no kernel/bootloader) | RHEL bootc (kernel + bootloader + systemd) |
| PID 1 | Application command | `systemd` |
| ENTRYPOINT | ✅ Yes | ❌ No (systemd handles init) |
| Deployment | Container runtime | Direct boot (bare metal/VM/cloud) |
| Immutable root | No | ✅ Yes (`/usr` read-only) |
| Writable directories | Any (depending on user) | Only `/etc` and `/var` |
| Use case | Run an application | Run a full operating system |

---

# Key Takeaways

- **bootc** = image mode for RHEL — full OS as a container image.
- **Build from a Containerfile** — base image already contains kernel + bootloader + systemd.
- **`--squash`** merges layers for efficient images.
- **No ENTRYPOINT** — systemd is the init process.
- **Immutable root** (`/usr`) — only `/etc` and `/var` are writable.
- **Move application data** to `/usr/share/` for immutability.
- **Testing** — run the bootc image as a container locally before pushing.
- **CI/CD integration** — build, test, and publish automatically.
- **Rollback** is instant — reboot to the previous image.
- **Image mode eliminates configuration drift** — consistent, secure, and reliable deployments.