# Creating and Managing Container Images — Notes

## 1. Container Images — The Hero of Containerization

- A container image is a **TAR archive** containing:
  - Binaries
  - Libraries
  - Configuration files
  - The application itself
- Packed with **metadata** (ENTRYPOINT, EXPOSE, LABEL, USER, etc.)
- **ENTRYPOINT** = the command invoked when a container starts from the image

---

## 2. Container Registries — Catalog and Authentication

### Registry UI:

```
catalog.redhat.com
```

### Registries:

| Registry | Authentication | Use Case |
|----------|----------------|----------|
| `registry.access.redhat.com` | ❌ Free | Public Red Hat images (UBI) |
| `registry.redhat.io` | ✅ Required | Customer images |
| `quay.io` | ✅ Free/Paid | Red Hat's container registry |

> 💡 **Best practice:** Use **service accounts/tokens**, not real passwords.

### Terms-based registry:

```
access.redhat.com/terms-based-registry
```

---

## 3. Registries Configuration — `registries.conf`

### File locations (precedence):

| Location | Scope |
|----------|-------|
| `~/.config/containers/registries.conf` | **User-specific** (highest priority) |
| `/etc/containers/registries.conf` | System-wide |
| `/etc/containers/registries.conf.d/*.conf` | Drop-in directory |

### View podman configuration:

```bash
podman info | grep registries -A 12
```

### Man page:

```bash
man containers-registries.conf
```

**Search for examples:**
```
/EXAMPLE
```

---

## 4. Blocking Registries — Example Configuration

### User `registries.conf`:

```bash
~/.config/containers/registries.conf
```

**Content:**
```
unqualified-search-registries = ["registry.access.redhat.com", "registry.redhat.io"]

[[registry]]
location = "docker.io"
blocked = true
```

### Testing blocked registry:

```bash
podman pull nginx
```

**Result:** ❌ **Denied** (docker.io is blocked)

> 💡 Blocking untrusted registries improves security.

---

## 5. Inspecting Images — `podman image inspect`

### Full metadata:

```bash
podman image inspect my_httpd:latest
```

### Extract specific data (JSON format):

```bash
podman image inspect my_httpd:latest --format "{{.Config.Entrypoint}}"
```

### Using `jq` (JSON parser):

```bash
podman image inspect my_httpd:latest | jq '.[].Config.Entrypoint'
```

> 💡 `jq` is a powerful JSON parser — works with any JSON data, not just podman.

### Install jq:

```bash
dnf install -y jq
```

---

## 6. Containerfile — Building Images

### Naming convention:

- **Containerfile** (capital C) — Red Hat preferred
- Dockerfile — legacy naming

### Build command:

```bash
podman build -t my_httpd:1.1.1 -f Containerfile .
```

| Option | Meaning |
|--------|---------|
| `-t` | **Tag** (image name + version) |
| `-f` | **File** (Containerfile to use) |
| `.` | **Build context** (current directory) |

> 💡 If you don't specify `-f`, podman looks for a file named `Containerfile`.

---

## 7. Containerfile Instructions — Common Directives

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image | `FROM registry.access.redhat.com/ubi9/ubi:latest` |
| `ENV` | Environment variable | `ENV HTTP_PORT=8080` |
| `RUN` | Command during build | `RUN dnf install -y httpd` |
| `WORKDIR` | Set working directory | `WORKDIR /var/www/html` |
| `COPY` | Copy files into image | `COPY index.html .` |
| `LABEL` | Metadata for the image | `LABEL maintainer="ricardo@redhat.com"` |
| `EXPOSE` | **Metadata** (does NOT open ports) | `EXPOSE 8080` |
| `USER` | User to run as | `USER 1001` |
| `ENTRYPOINT` | Command when container starts | `ENTRYPOINT ["httpd", "-DFOREGROUND"]` |

> ⚠️ **EXPOSE does not open ports** — it's just metadata for the user and runtime.

---

## 8. Multi-Line RUN Commands

### Using backslash (`\`) for readability:

```
RUN dnf install -y httpd && \
    dnf clean all && \
    rm -rf /var/cache/dnf && \
    sed -i 's/^Listen 80/Listen ${HTTP_PORT}/' /etc/httpd/conf/httpd.conf && \
    sed -i 's/^#ServerName.*/ServerName localhost/' /etc/httpd/conf/httpd.conf && \
    sed -i 's/^ErrorLog .*/ErrorLog "\/dev\/stderr"/' /etc/httpd/conf/httpd.conf && \
    sed -i 's/^CustomLog .*/CustomLog "\/dev\/stdout" common/' /etc/httpd/conf/httpd.conf
```

> 💡 Backslash (`\`) continues the command to the next line for readability.

---

## 9. UBI — Universal Base Image

- **Free for everyone** to use
- No subscription needed
- Supported by Red Hat
- Regularly updated (free of known CVEs)

### Example:

```
FROM registry.access.redhat.com/ubi9/ubi:latest
```

> 💡 **UBI9** → RHEL 9-based. **UBI10** → RHEL 10-based.

---

## 10. Tagging Images — `podman tag`

### Create a new tag for an existing image:

```bash
podman tag my_httpd:1.1.1 quay.io/rdacosta/my_httpd:latest
```

### View images:

```bash
podman images
```

**Output:**
```
REPOSITORY                      TAG       IMAGE ID
quay.io/rdacosta/my_httpd       latest    93echo...
quay.io/rdacosta/my_httpd       1.1.1     93echo...
```

> 💡 Same `IMAGE ID` = same image, multiple tags.

---

## 11. Pushing Images — `podman push`

### Login to registry (Quay.io):

```bash
podman login -u rdacosta+bender -p <token> quay.io
```

### Push image:

```bash
podman push quay.io/rdacosta/my_httpd:latest
```

> 💡 **Best practice:** Use **Robot accounts** (Quay) or **Service accounts** (Red Hat) instead of real passwords.

---

## 12. Pruning Dangling Images — `podman image prune`

- **Dangling images** = images not associated with any container
- They consume local storage

### Remove all unused images:

```bash
podman image prune --all
```

### View before pruning:

```bash
podman images
```

### View containers using images:

```bash
podman ps -a
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View podman configuration | `podman info` |
| View registries config | `man containers-registries.conf` |
| Inspect image metadata | `podman image inspect my_httpd:latest` |
| Extract Entrypoint | `podman inspect my_httpd:latest --format "{{.Config.Entrypoint}}"` |
| Extract with jq | `podman image inspect my_httpd:latest \| jq '.[].Config.Entrypoint'` |
| Build image | `podman build -t my_httpd:1.1.1 -f Containerfile .` |
| Tag image | `podman tag my_httpd:1.1.1 quay.io/user/my_httpd:latest` |
| Login to registry | `podman login quay.io` |
| Push image | `podman push quay.io/user/my_httpd:latest` |
| Prune dangling images | `podman image prune --all` |

---

# Containerfile — Complete Example

```
FROM registry.access.redhat.com/ubi9/ubi:latest

ENV HTTP_PORT=8080

RUN dnf install -y httpd && \
    dnf clean all && \
    rm -rf /var/cache/dnf && \
    sed -i 's/^Listen 80/Listen ${HTTP_PORT}/' /etc/httpd/conf/httpd.conf && \
    sed -i 's/^#ServerName.*/ServerName localhost/' /etc/httpd/conf/httpd.conf && \
    sed -i 's/^ErrorLog .*/ErrorLog "\/dev\/stderr"/' /etc/httpd/conf/httpd.conf && \
    sed -i 's/^CustomLog .*/CustomLog "\/dev\/stdout" common/' /etc/httpd/conf/httpd.conf

WORKDIR /var/www/html
COPY index.html .

LABEL maintainer="ricardo@redhat.com" \
      version="1.0"

EXPOSE 8080

USER 1001

ENTRYPOINT ["httpd", "-DFOREGROUND"]
```

---

# Key Takeaways

- **Containerfile** = instructions for building a container image.
- **FROM** = base image (use UBI for free, supported images).
- **RUN** = commands executed during build.
- **ENV** = environment variables (can be used in RUN commands).
- **COPY** = copies files from build context into the image.
- **EXPOSE** = **metadata only** — does NOT open ports.
- **ENTRYPOINT** = command that runs when the container starts.
- **USER** = run container with least privilege (use `1001`, not root).
- **UBI** = Universal Base Image — free for everyone.
- **Regularly rebuild images** to stay current with security updates.
- **`podman tag`** = create additional tags for the same image.
- **`podman push`** = upload image to a registry.
- **Robot accounts** (Quay) or **Service accounts** (Red Hat) = secure authentication.