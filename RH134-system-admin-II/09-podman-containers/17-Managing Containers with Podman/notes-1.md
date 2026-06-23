# Essential Container Definitions — Notes

## 1. Core Definitions

### Container:

- A **running instance** of a **container image**

### Container Image:

- A **single file** (actually a TAR archive)
- Contains all files needed to support an application:
  - Binaries
  - Libraries
  - Configuration files
  - The application itself
- Packed with **metadata** (including `ENTRYPOINT`)

### Container Image Contents:

```
Container Image (TAR archive)
├── Binaries
├── Libraries
├── Config files
└── Application
```

> 💡 "Container" = running instance. "Container image" = the blueprint.

---

## 2. Container Registries

| Registry | Authentication | Use Case |
|----------|----------------|----------|
| `registry.access.redhat.com` | ❌ Free | Public Red Hat images |
| `registry.redhat.io` | ✅ Required | Red Hat customer images |
| `quay.io` | ✅ Free/Paid | Red Hat's container registry |

> 💡 `quay.io` is pronounced "key.io" (not "kway.io").

---

## 3. Container Runtime

- Software needed to run containers on a **container host**
- Also called a **container engine**

### RHEL Container Runtimes:

| Runtime | Purpose |
|---------|---------|
| **podman** | Main container runtime (RHEL default) |
| **crun** | Underlying OCI runtime (replaces `runc`) |
| **cri-o** | Kubernetes/OpenShift runtime (OCI-compliant) |

> 💡 **Podman** is the container runtime on RHEL. Docker is **not** distributed with RHEL.

---

## 4. Virtual Machines vs Containers

| Feature | Virtual Machine | Container |
|---------|-----------------|-----------|
| **Image type** | VM image (qcow2, vmdk, raw) | Container image (OCI) |
| **Size** | Large (full OS) | Small (only app + dependencies) |
| **Bootloader** | ✅ Yes (GRUB) | ❌ No |
| **Kernel** | ✅ Yes (full OS kernel) | ❌ No (shares host kernel) |
| **Init system** | ✅ Yes (systemd) | Optional (only if needed) |
| **Hardware access** | ✅ Low-level (kernel) | ❌ No (no kernel) |
| **Resource usage** | High (many processes) | Low (few processes) |
| **Portability** | Hypervisor-dependent | OCI standard (any OCI engine) |

> 💡 **Containers share the host kernel** — they don't have their own.

---

## 5. OCI — Open Containers Initiative

- **Standard** for containers
- Any OCI-compliant image can run on any OCI-compliant runtime
- Ensures **portability** across different container engines

```
OCI Image → Any OCI Runtime → Container
```

---

## 6. When to Use Virtual Machines vs Containers

### Use VMs when:

- Need full operating system isolation
- Need low-level hardware access (e.g., packet capturing)
- Running legacy applications not container-ready

### Use Containers when:

- Want lightweight, fast deployment
- Need portability across environments
- Running microservices
- Want efficient resource usage

---

## 7. The Container Architecture Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    Container Host                           │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                   Container Runtime                   │   │
│  │                     (podman)                          │   │
│  │                       │                               │   │
│  │                       ▼                               │   │
│  │    ┌─────────────┐  ┌─────────────┐                  │   │
│  │    │  Container  │  │  Container  │                  │   │
│  │    │  MySQL 5.7  │  │  MySQL 8.0  │                  │   │
│  │    │  (running)  │  │  (running)  │                  │   │
│  │    └─────────────┘  └─────────────┘                  │   │
│  └──────────────────────────────────────────────────────┘   │
│                              ▲                               │
│                              │                               │
│                    ┌─────────┴─────────┐                    │
│                    │  Container Image  │                    │
│                    │  (MySQL 5.7)      │                    │
│                    └─────────┬─────────┘                    │
│                              │                               │
│                    ┌─────────┴─────────┐                    │
│                    │  Image Registry   │                    │
│                    │  quay.io /        │                    │
│                    │  registry.redhat  │                    │
│                    └───────────────────┘                    │
└─────────────────────────────────────────────────────────────┘
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Install Podman | `dnf install -y podman` |
| Pull image from registry | `podman pull registry.redhat.io/ubi9` |
| List images | `podman images` |
| Run container | `podman run -it ubi9 /bin/bash` |
| List running containers | `podman ps` |
| List all containers | `podman ps -a` |

---

# Key Takeaways

- **Container** = running instance of a container image.
- **Container image** = TAR archive with app + dependencies + metadata.
- **Image Registry** = stores container images (e.g., `quay.io`, `registry.redhat.io`).
- **Container Runtime** = software that runs containers (RHEL uses **podman**).
- **OCI** = Open Containers Initiative — standard for container images and runtimes.
- **Virtual machines** have full OS (kernel, bootloader, init system) → larger, more resources.
- **Containers** share the host kernel → smaller, faster, more efficient.
- **Containers cannot** access hardware directly (no kernel).
- **Podman** is the default container runtime on RHEL (Docker is not distributed).
- **`crun`** is the underlying OCI runtime used by Podman.
- **`cri-o`** is used by Kubernetes distributions (OpenShift).