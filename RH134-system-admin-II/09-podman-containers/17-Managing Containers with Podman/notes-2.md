# Running Containers with Podman — Notes

## 1. Podman Basics — Viewing Containers and Images

### List running containers:

```bash
podman ps
```

### List all containers (including stopped):

```bash
podman ps -a
```

### List downloaded container images:

```bash
podman images
```

### Check Podman version:

```bash
podman version
```

> 💡 **Tab completion is your friend:** Type `podman` + space + double-tab to see all subcommands.

---

## 2. Container Registries — Authentication

### Registry authentication using service account:

```bash
podman login registry.redhat.io
```

### Generate a token:

```
access.redhat.com/terms-based-registry
```

### Credential storage location:

```bash
$XDG_RUNTIME_DIR/containers/auth.json
```

### View decoded credentials:

```bash
cat $XDG_RUNTIME_DIR/containers/auth.json | base64 -d
```

> ⚠️ **Security warning:** Credentials are stored in plaintext (Base64). Use **service account tokens**, not your real password.

---

## 3. Registries Configuration — `registries.conf`

### Global configuration:

```
/etc/containers/registries.conf
```

### Drop-in directory:

```
/etc/containers/registries.conf.d/*.conf
```

### User configuration:

```
~/.config/containers/registries.conf
```

### View man page:

```bash
man containers-registries.conf
```

**Search for examples:**
```
/EXAMPLE
```

---

## 4. Container Image Naming

```
quay.io/rdacosta/my_httpd:latest
   │        │         │       │
   │        │         │       └── Tag (version)
   │        │         └────────── Image name
   │        └──────────────────── Namespace (user/organization)
   └───────────────────────────── Registry
```

| Part | Example | Meaning |
|------|---------|---------|
| Registry | `quay.io` | Image registry |
| Namespace | `rdacosta` | User/organization |
| Image | `my_httpd` | Image name |
| Tag | `latest` | Version identifier |

---

## 5. Running a Container — `podman run`

### Basic syntax:

```bash
podman run [OPTIONS] IMAGE [COMMAND]
```

### Example — run a web server container:

```bash
podman run -d -p 8080:8080 quay.io/rdacosta/my_httpd:latest
```

| Option | Meaning |
|--------|---------|
| `-d` | **Detached** — run in background |
| `-p host:container` | **Port forwarding** — map container port to host |

### Image download behavior:

- If the image is **not** downloaded locally, Podman **automatically pulls** it from the registry

---

## 6. Port Forwarding — `-p`

```
-p 8080:8080
    │       │
    │       └── Container port (inside container)
    └─────────── Host port (on your system)
```

> 💡 Traffic to `localhost:8080` is forwarded to the container on port `8080`.

---

## 7. Viewing Container Logs — `podman logs`

```bash
podman logs <CONTAINER_ID>
```

or

```bash
podman logs <CONTAINER_NAME>
```

> 💡 Useful for troubleshooting application issues.

---

## 8. Stopping and Starting Containers

### Stop a running container:

```bash
podman stop <CONTAINER_ID>
```

### Start a stopped container:

```bash
podman start <CONTAINER_ID>
```

### Restart a container:

```bash
podman restart <CONTAINER_ID>
```

---

## 9. Removing Containers — `podman rm`

### Remove a stopped container:

```bash
podman rm <CONTAINER_ID>
```

### Remove a running container (force):

```bash
podman rm --force <CONTAINER_ID>
```

> ⚠️ `--force` stops and removes the container in one command.

---

## 10. Removing Images — `podman rmi`

### Remove a container image:

```bash
podman rmi <IMAGE_NAME>
```

or

```bash
podman rmi <IMAGE_ID>
```

> 💡 You cannot remove an image while a container using it is still present.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| List running containers | `podman ps` |
| List all containers | `podman ps -a` |
| List images | `podman images` |
| Login to registry | `podman login registry.redhat.io` |
| Run container (detached) | `podman run -d -p 8080:8080 image` |
| Stop container | `podman stop container` |
| Start container | `podman start container` |
| Restart container | `podman restart container` |
| View container logs | `podman logs container` |
| Remove container | `podman rm container` |
| Force remove container | `podman rm --force container` |
| Remove image | `podman rmi image` |
| View Podman run man page | `man podman-run` |

---

# Container Lifecycle — Visual Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Image in Registry (e.g., quay.io)                       │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼ podman pull (or run)
┌─────────────────────────────────────────────────────────────┐
│ 2. Image downloaded to local system                        │
│    └── podman images                                        │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼ podman run
┌─────────────────────────────────────────────────────────────┐
│ 3. Container running                                        │
│    └── podman ps                                            │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼ podman stop
┌─────────────────────────────────────────────────────────────┐
│ 4. Container stopped (exited)                               │
│    └── podman ps -a                                         │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼ podman rm
┌─────────────────────────────────────────────────────────────┐
│ 5. Container removed                                        │
│    └── podman ps -a (container gone)                        │
└─────────────────────────────────────────────────────────────┘
```

---

# Key Takeaways

- **`podman ps`** = show running containers.
- **`podman ps -a`** = show all containers (including stopped).
- **`podman images`** = show downloaded container images.
- **`podman login`** = authenticate to image registry (use **tokens**, not real passwords).
- **Credentials** are stored in `$XDG_RUNTIME_DIR/containers/auth.json` (Base64 encoded).
- **`podman run -d -p HOST:CONTAINER IMAGE`** = run container in background with port forwarding.
- **Container image naming:** `registry/namespace/image:tag`.
- **`podman logs`** = view container application output.
- **`podman stop`** / **`podman start`** = stop/start containers.
- **`podman rm`** = remove stopped containers.
- **`podman rm --force`** = stop and remove running containers.
- **`podman rmi`** = remove container images (image must not be in use).
- **Tab completion** and **man pages** are your best friends.