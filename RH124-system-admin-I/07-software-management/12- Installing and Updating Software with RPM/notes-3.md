# Managing Software Repositories — Notes

## 1. What Are DNF Repositories?

- **Repositories** = curated collections of RPM packages
- Hosted on: websites, FTP servers, or local file systems
- RHEL includes default configuration for Red Hat repositories
- You may need to add **third-party** or **organization-specific** repositories (e.g., Adobe, EPEL)

---

## 2. Default Red Hat Repositories

### List all repositories with status:

```bash
dnf repolist all
```

**Output example:**

| Repo ID | Repo Name | Status |
|---------|-----------|--------|
| `rhel-10-for-x86_64-baseos-rpms` | RHEL 10 - BaseOS (RPMs) | enabled |
| `rhel-10-for-x86_64-appstream-rpms` | RHEL 10 - AppStream (RPMs) | enabled |

> 💡 **BaseOS** = core RHEL components. **AppStream** = applications, runtimes, and tools.

---

## 3. Simple Content Access (SCA)

### What SCA does:

- Systems can access **any repository** from any subscription you own
- **No need** to attach subscriptions per system
- Enable SCA on:
  - Red Hat Customer Portal (My Subscriptions → Subscription Allocations)
  - Red Hat Satellite server

| Without SCA | With SCA |
|-------------|----------|
| Attach subscription per system | No per-system attachment |
| Manual tracking of entitlements | Simplified access |
| Risk of over/under-subscription | Access all repositories automatically |

---

## 4. Enabling/Disabling Repositories — `dnf config-manager`

### Enable a repository:

```bash
dnf config-manager --enable rhel-10-for-x86_64-baseos-debug-rpms
```

### Disable a repository:

```bash
dnf config-manager --disable rhel-10-for-x86_64-baseos-debug-rpms
```

### Temporary enable (just for one command):

```bash
dnf --enablerepo=epel install package-name
```

### Temporary disable (just for one command):

```bash
dnf --disablerepo='*' --enablerepo=local-repo install package-name
```

> 💡 `--disablerepo='*'` disables **all** repositories, then selectively enable.

---

## 5. Repository Configuration Files — `.repo` Files

### Location:

```bash
/etc/yum.repos.d/*.repo
```

### Alternative (not recommended):

```bash
/etc/dnf/dnf.conf
```

> 📌 **Best practice:** Use `.repo` files in `/etc/yum.repos.d/`. Reserve `dnf.conf` for additional configurations.

### Precedence:

`.repo` files take precedence over settings in `dnf.conf`.

---

## 6. Manual Repository Configuration — `.repo` File Example

### Create a `.repo` file:

```bash
sudo vi /etc/yum.repos.d/epel.repo
```

### Example content:

```ini
[EPEL]
name=EPEL 10
baseurl=https://dl.fedoraproject.org/pub/epel/10/Everything/x86_64/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-10
```

### `.repo` file parameters:

| Parameter | Meaning | Example |
|-----------|---------|---------|
| `[repo-id]` | Unique repository identifier | `[EPEL]` |
| `name` | Human-readable description | `name=EPEL 10` |
| `baseurl` | URL to repository | `baseurl=https://...` |
| `enabled` | 1 = enabled, 0 = disabled | `enabled=1` |
| `gpgcheck` | 1 = verify signatures, 0 = skip | `gpgcheck=1` |
| `gpgkey` | URL to GPG public key | `gpgkey=file:///path/to/key` |

---

## 7. Adding Repositories with `dnf config-manager --add-repo`

### Command:

```bash
dnf config-manager --add-repo="https://dl.fedoraproject.org/pub/epel/10/Everything/x86_64/"
```

### What happens:

1. Creates a `.repo` file in `/etc/yum.repos.d/`
2. File name format: `{domain}_path_.repo`

### Resulting file:

```bash
cat /etc/yum.repos.d/dl.fedoraproject.org_pub_epel_10_Everything_x86_64_.repo
```

```ini
[dl.fedoraproject.org_pub_epel_10_Everything_x86_64_]
name=created by dnf config-manager from https://dl.fedoraproject.org/pub/epel/10/Everything/x86_64/
baseurl=https://dl.fedoraproject.org/pub/epel/10/Everything/x86_64/
enabled=1
```

> 💡 You can edit this file to add `gpgcheck=1` and `gpgkey=` parameters manually.

---

## 8. GPG Key Verification — Why It Matters

### The risk:

- Without GPG verification, you could install **compromised or forged packages**

### The process:

1. Repository provides GPG public key
2. RPM packages are signed with matching **private key**
3. `rpm`/`dnf` uses **public key** to verify signature
4. If verification fails → package is not trusted

### Import a GPG key:

```bash
rpm --import https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-10
```

### Local key storage:

```bash
/etc/pki/rpm-gpg/
```

### Example `.repo` file with local key reference:

```ini
[EPEL]
name=EPEL 10
baseurl=https://dl.fedoraproject.org/pub/epel/10/Everything/x86_64/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-10
```

---

## 9. Installing Repositories via RPM Packages

### Some repositories provide an RPM package that:

- Installs the `.repo` file
- Installs the GPG public key

### Example — EPEL repository:

```bash
# Step 1: Import GPG key
rpm --import https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-10

# Step 2: Install repository RPM
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-10.noarch.rpm
```

> ⚠️ **Warning:** Import the GPG key **before** installing signed packages. Otherwise, `dnf` will fail to install them.

---

## 10. The `--nogpgcheck` Option — Dangerous!

```bash
dnf install --nogpgcheck some-package.rpm
```

| Pros | Cons |
|------|------|
| Quick test of packages | **No verification** of package authenticity |
| Bypasses missing keys | Risk of installing **compromised** or **forged** packages |

> ⚠️ **Never use `--nogpgcheck` in production.**

---

## 11. Multiple Repositories in One File

### Example: `/etc/yum.repos.d/epel.repo`

```ini
[epel]
name=Extra Packages for Enterprise Linux $releasever - $basearch
baseurl=https://download.example/pub/epel/$releasever/Everything/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-$releasever

[epel-source]
name=Extra Packages for Enterprise Linux $releasever - $basearch - Source
baseurl=https://download.example/pub/epel/$releasever/Everything/source/tree/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-$releasever
```

### Variables in `.repo` files:

| Variable | Meaning |
|----------|---------|
| `$releasever` | RHEL release version (e.g., `10`) |
| `$basearch` | System architecture (e.g., `x86_64`) |
| `$infra` | Infrastructure type |
| `$contentdir` | Content directory |

> 💡 Repositories with `enabled=0` are **disabled by default** but can be temporarily enabled with `--enablerepo`.

---

## 12. Persistence vs. Temporary Options

| Method | Persistence |
|--------|-------------|
| `dnf config-manager --enable` | **Persistent** (writes to `.repo` file) |
| `dnf config-manager --disable` | **Persistent** (writes to `.repo` file) |
| `dnf --enablerepo=pattern` | **Temporary** (just for that command) |
| `dnf --disablerepo=pattern` | **Temporary** (just for that command) |

### Example — temporary enable:

```bash
dnf --enablerepo=epel install ansible
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| List all repos with status | `dnf repolist all` |
| List only enabled repos | `dnf repolist` |
| Enable a repository (persistent) | `dnf config-manager --enable repo-id` |
| Disable a repository (persistent) | `dnf config-manager --disable repo-id` |
| Add repository from URL | `dnf config-manager --add-repo="URL"` |
| Temporarily enable repo | `dnf --enablerepo=repo-id install pkg` |
| Temporarily disable repo | `dnf --disablerepo=repo-id update` |
| Temporarily disable all repos | `dnf --disablerepo='*' install pkg` |
| Import GPG key | `rpm --import https://url/to/key` |
| Install repository RPM | `dnf install https://url/to/repo.rpm` |
| Install without GPG check (dangerous) | `dnf --nogpgcheck install pkg.rpm` |

---

# `.repo` File Parameters Cheat Sheet

| Parameter | Required? | Purpose | Example |
|-----------|-----------|---------|---------|
| `[repo-id]` | Yes | Unique identifier (no spaces) | `[EPEL]` |
| `name` | Yes | Human-readable description | `name=EPEL 10` |
| `baseurl` | Yes* | URL to repository | `baseurl=https://...` |
| `enabled` | No (default=0) | 1=enabled, 0=disabled | `enabled=1` |
| `gpgcheck` | No (default=0) | 1=verify signatures | `gpgcheck=1` |
| `gpgkey` | If gpgcheck=1 | URL to GPG public key | `gpgkey=file:///path` |
| `metalink` | Alternative to baseurl | Metalink URL | `metalink=https://...` |

> *Either `baseurl` or `metalink` is required.

---

# GPG Key Verification Flow

```
1. Repository provides GPG public key
              ↓
2. Administrator imports key: rpm --import <key-url>
              ↓
3. Repository signs packages with private key
              ↓
4. dnf downloads package + signature
              ↓
5. dnf verifies signature using public key
              ↓
6. If valid → install. If invalid → reject.
```

---

# Key Takeaways

- **`dnf repolist all`** shows all repositories and their enabled/disabled status.
- **Simple Content Access (SCA)** eliminates per-system subscription attachment.
- **`dnf config-manager --enable/--disable`** makes persistent changes.
- **`--enablerepo` / `--disablerepo`** are **temporary** (single command only).
- **`.repo` files** in `/etc/yum.repos.d/` define repository configuration.
- **`gpgcheck=1`** is critical for security — verifies package authenticity.
- **Import GPG keys** with `rpm --import` **before** using the repository.
- **`--nogpgcheck` is dangerous** — never use in production.
- **Repository RPM packages** simplify configuration (e.g., `epel-release`).
- **`enabled=0`** disables a repository by default — enable with `--enablerepo`.
- **Temporary overrides** are useful for one-off installations without permanently enabling repos.