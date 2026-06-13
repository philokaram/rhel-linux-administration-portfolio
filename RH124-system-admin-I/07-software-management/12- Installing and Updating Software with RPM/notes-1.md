# RPM Software Packages — Notes

## 1. What Is RPM?

- **RPM** = Red Hat Package Manager (originally created by Red Hat)
- Today: **Standard way** software is packaged, shared, and installed across RHEL

### What RPM does:

| Benefit | Description |
|---------|-------------|
| Bundles software | Instead of scattering files manually |
| Tracks files | Knows exactly which files a package installs |
| Clean removal | Knows which files to remove on uninstall |
| Dependency tracking | Knows what supporting software is required |

> 💡 Makes life easy for administrators.

---

## 2. The RPM Database — Local Record Keeper

Every RHEL system has a **local RPM database** that tracks:

| Information Tracked | Example |
|---------------------|---------|
| What is installed | Package names |
| What version | `wget-1.21-17.el9.x86_64` |
| What architecture | `x86_64`, `noarch` |

### Where packages come from:

```
RPM Repository → RPM File → RPM Database → Installed Software
```

---

## 3. RPM File Naming Convention

### Example:

```
wget-1.21-17.el9.x86_64.rpm
```

| Part | Meaning | Example |
|------|---------|---------|
| Name | Software name | `wget` |
| Version | Software version | `1.21` |
| Release | Package release number | `17.el9` |
| Architecture | CPU architecture | `x86_64` |

### Common architectures:

| Architecture | Meaning |
|--------------|---------|
| `x86_64` | 64-bit Intel/AMD |
| `aarch64` | 64-bit ARM |
| `noarch` | Works on any architecture |
| `src` | Source RPM (contains source code) |

---

## 4. Two Commands for RPM Management

| Feature | `rpm` command | `dnf` command |
|---------|---------------|---------------|
| Install local `.rpm` file | ✅ Yes | ✅ Yes |
| Query RPM database | ✅ Yes | ✅ Yes |
| Query RPM repositories | ❌ No | ✅ Yes |
| Dependency resolution | ❌ No | ✅ Yes |

> 💡 `dnf` is more powerful — it works with **repositories**.
> 💡 `rpm` is lower-level — good for inspecting and local installs.

---

## 5. Inside an RPM File — It's an Archive!

### Extract an RPM to see its contents:

```bash
rpm2cpio package.rpm | cpio -duim
```

### `cpio` options:

| Option | Meaning |
|--------|---------|
| `-d` | Create directories as needed |
| `-u` | Replace older files |
| `-i` | Extract (input mode) |
| `-m` | Preserve modification times |

### Example (wget package):

```bash
rpm2cpio wget-1.21-17.el9.x86_64.rpm | cpio -duim
```

### Typical contents:

```
.
├── etc/
│   └── wgetrc          # Configuration file
└── usr/
    ├── bin/
    │   └── wget        # Binary/executable
    ├── lib/
    │   └── ...         # Libraries
    └── share/
        └── man/        # Documentation
```

---

## 6. Querying RPMs — The `rpm -q` Family

### Query a package (must be installed):

```bash
rpm -q wget
```

**Output:** `wget-1.21-17.el9.x86_64`

### Query an RPM file (not installed):

```bash
rpm -qpl wget-1.21-17.el9.x86_64.rpm
```

| Option | Meaning |
|--------|---------|
| `-q` | Query |
| `-p` | Query an **uninstalled** package file |
| `-l` | List all files in the package |

### Common query combinations:

| Command | Purpose |
|---------|---------|
| `rpm -ql wget` | List files (installed package) |
| `rpm -qpl package.rpm` | List files (uninstalled package) |
| `rpm -qc wget` | List configuration files |
| `rpm -qd wget` | List documentation files |
| `rpm -qi wget` | Show package information |
| `rpm -qf /path/to/file` | Find which package owns a file |

### Find which package owns a file:

```bash
rpm -qf /etc/wgetrc
```

**Output:** `wget-1.21-17.el9.x86_64`

> ⚠️ This only works for **installed** packages.

---

## 7. Installing RPMs with `rpm`

### Basic install:

```bash
rpm -ivh package.rpm
```

| Option | Meaning |
|--------|---------|
| `-i` | Install |
| `-v` | Verbose (show details) |
| `-h` | Hash marks (progress bar) |

### Limitations of `rpm -i`:

- ❌ **No dependency resolution**
- If `nmap` requires `libxyz`, you must install `libxyz` first manually

> 💡 For dependency resolution, use `dnf install package.rpm`.

---

## 8. Package Scripts (from Source RPM)

Behind the scenes, RPMs can contain scripts that run at certain times:

| Script | When it runs |
|--------|--------------|
| `%pre` | Before installation |
| `%post` | After installation |
| `%preun` | Before uninstallation |
| `%postun` | After uninstallation |

> 📌 These are only visible in **source RPMs** (`.src.rpm`).

---

## 9. Finding Which Package Provides a File — The Better Way

### With `rpm` (installed files only):

```bash
rpm -qf /etc/wgetrc
```

### With `dnf` (even for not-yet-installed packages):

```bash
dnf provides /etc/wgetrc
```

or

```bash
dnf whatprovides /etc/wgetrc
```

> 💡 `dnf` can search repositories — `rpm` cannot.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| List files in installed package | `rpm -ql wget` |
| List files in RPM file (not installed) | `rpm -qpl package.rpm` |
| List config files | `rpm -qc wget` |
| List docs | `rpm -qd wget` |
| Show package info | `rpm -qi wget` |
| Find package owning a file (installed) | `rpm -qf /path/to/file` |
| Find package owning a file (any) | `dnf provides /path/to/file` |
| Install RPM (no deps) | `rpm -ivh package.rpm` |
| Extract RPM contents | `rpm2cpio package.rpm | cpio -duim` |
| Check if package is installed | `rpm -q wget` |

---

# RPM Query Options Cheat Sheet

| Option Group | Option | Meaning |
|--------------|--------|---------|
| **Query type** | `-q` | Query |
| | `-qi` | Query info |
| | `-ql` | Query list (files) |
| | `-qc` | Query config files |
| | `-qd` | Query doc files |
| | `-qf` | Query file (which package owns this) |
| **Target** | (no extra) | Installed package |
| | `-p` | Package file (`.rpm`) |
| **Verbosity** | `-v` | Verbose |
| | `-h` | Hash marks (progress) |

### Examples:

```bash
rpm -ql wget                    # Installed package, list files
rpm -qpl wget-1.21.rpm          # Uninstalled file, list files
rpm -qf /etc/wgetrc             # Find owner of installed file
```

---

# Architecture of RPM Management

```
┌─────────────────────────────────────────────────────────┐
│                    Red Hat CDN / Repository              │
│              (curated collection of RPMs)                │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                      RPM File                            │
│              wget-1.21-17.el9.x86_64.rpm                 │
│  ┌─────────────────────────────────────────────────┐    │
│  │ • Binaries  • Libraries  • Configs  • Docs       │    │
│  │ • Scripts (%pre, %post, %preun, %postun)        │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                    RPM Database                          │
│         (tracks everything installed on system)          │
│  ┌─────────────────────────────────────────────────┐    │
│  │ Package: wget-1.21-17.el9.x86_64                 │    │
│  │ Files: /usr/bin/wget, /etc/wgetrc, ...          │    │
│  │ Size: 1.2 MB                                     │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

---

# Key Takeaways

- **RPM** = building block of RHEL software management.
- **RPM database** = single source of truth for installed packages.
- **RPM files are archives** — you can extract them to see what they contain.
- **`rpm` command** = low-level, works with local files and database, but **no dependency resolution**.
- **`dnf` command** = higher-level, works with repositories, **handles dependencies**.
- Use `rpm -qf` to find which **installed** package owns a file.
- Use `dnf provides` to find which package (even **not installed**) provides a file.
- **Naming convention:** `name-version-release.architecture.rpm`
- Traceability + consistency = why RPM matters in enterprise Linux.