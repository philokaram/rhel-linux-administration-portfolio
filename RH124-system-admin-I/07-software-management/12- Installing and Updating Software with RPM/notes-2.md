# Keeping RHEL Secure and Up-to-Date — Notes

## 1. Prerequisite: System Registration

### Check registration status:

```bash
subscription-manager identity    # More reliable than status
```

### If not registered:

| Method | Command |
|--------|---------|
| Modern | `rhc connect --username ...` |
| Traditional | `subscription-manager register --username ...` |

### For air-gapped environments — Satellite:

- **Satellite** proxies content from Red Hat CDN
- Satellite needs internet access
- **Your servers** only need access to Satellite

```
Internet → Satellite (downloads once) → Your Servers
```

---

## 2. Red Hat Repositories

### List enabled repositories:

```bash
subscription-manager repos --list-enabled
```

### Two main repositories:

| Repository | Content |
|------------|---------|
| `rhel-10-for-x86_64-baseos-rpms` | Core RHEL components |
| `rhel-10-for-x86_64-appstream-rpms` | Application runtimes, languages, web servers, databases |

---

## 3. Searching for Software — `dnf search`

```bash
dnf search nodejs
```

**What it does:** Searches enabled repositories for packages matching the keyword.

---

## 4. Listing Available Versions

### Show all versions (including duplicates):

```bash
dnf list --showduplicates nodejs
```

**Output shows:** Version numbers and which repository they come from.

---

## 5. Installing Software — `dnf install`

### Install latest version:

```bash
dnf install nodejs
```

### Automatic dependency resolution:

- DNF will **automatically** identify and install required dependencies
- Prompts for confirmation before proceeding

### Install without prompt (`-y`):

```bash
dnf install -y httpd
```

### What happens:

```
Installing:
  httpd
  httpd-core
  httpd-filesystem
  httpd-tools
  ... (dependencies)
```

---

## 6. Package Information — `dnf info`

```bash
dnf info nodejs
```

**Shows:** Name, version, architecture, repository, size, description, and whether installed.

---

## 7. Errata — Updates Classification

Updates come in three types:

| Errata Type | Meaning | Example ID |
|-------------|---------|------------|
| **RHBA** | Bug fix | `RHBA-2025:9419` |
| **RHEA** | Enhancement | `RHEA-2025:xxxx` |
| **RHSA** | Security Advisory | `RHSA-2025:0960` |

### Security terminology:

| Term | Meaning |
|------|---------|
| **CVE** | Common Vulnerabilities and Exposures (vendor-neutral ID) |
| **CVSS score** | Severity rating (0-10) |
| **CVSS severity** | 0.1-3.9 = Low, 4.0-6.9 = Moderate, 7.0-8.9 = Important, 9.0-10 = Critical |

> 💡 One RHSA can address **multiple CVEs**.

---

## 8. Viewing Available Updates — `dnf updateinfo`

### List all update information:

```bash
dnf updateinfo list
```

**Output example:**
```
RHSA-2025:0960 Important/sudo security update
RHBA-2025:9419 bugfix/bash bug fix update
RHEA-2025:xxxx enhancement/curl enhancement update
```

### Filter by type:

```bash
dnf updateinfo list | grep RHSA    # Security only
dnf updateinfo list | grep RHBA    # Bug fixes only
dnf updateinfo list | grep RHEA    # Enhancements only
```

### Get detailed info about a specific advisory:

```bash
dnf updateinfo info RHSA-2025:0960
```

**Shows:**
- Update ID
- Type (security)
- Severity (Important/Critical)
- CVEs addressed (e.g., CVE-2025-32462)
- Description
- Affected packages

---

## 9. Installing Updates — `dnf update`

### Update everything:

```bash
dnf update
```

### Update only security updates:

```bash
dnf update --security
```

### Update only specific severity levels:

```bash
dnf update --sec-severity=Important --sec-severity=Critical
```

> 💡 This limits updates to **highest priority** security fixes.

---

## 10. Checking If Reboot Is Required

```bash
dnf needs-restarting -r
```

| Output | Meaning |
|--------|---------|
| (no output) | No reboot needed |
| `kernel` or `kernel-core` listed | Reboot required to fully utilize updates |

---

## 11. Removing Software — `dnf remove`

```bash
dnf remove httpd
```

**What happens:**
- Removes `httpd`
- Removes **dependencies** that were installed with it (if no longer needed)
- Prompts for confirmation

---

## 12. Transaction History — `dnf history`

### View history:

```bash
dnf history
```

**Shows:** Transaction ID, date/time, action (install/update/remove), number of packages affected.

### Undo a transaction:

```bash
dnf history undo <transaction_id>
```

**Example:**

```bash
dnf history undo 11    # Re-installs httpd if that transaction removed it
```

---

## 13. Package Groups

### List available groups:

```bash
dnf group list
```

### View group details:

```bash
dnf group info "Development Tools"
```

**Shows:**
- Mandatory packages
- Default packages
- Optional packages

### Install a group:

```bash
dnf group install "Development Tools"
```

> 💡 Use `Esc + .` to reuse the last argument after typing `dnf group install`

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Check registration | `subscription-manager identity` |
| List enabled repos | `subscription-manager repos --list-enabled` |
| Search for a package | `dnf search nodejs` |
| List versions (with duplicates) | `dnf list --showduplicates nodejs` |
| Install a package | `dnf install nodejs` |
| Install without prompt | `dnf install -y httpd` |
| Get package info | `dnf info nodejs` |
| List available updates | `dnf updateinfo list` |
| Get advisory details | `dnf updateinfo info RHSA-2025:0960` |
| Update everything | `dnf update` |
| Update security only | `dnf update --security` |
| Update by severity | `dnf update --sec-severity=Important --sec-severity=Critical` |
| Check reboot needed | `dnf needs-restarting -r` |
| Remove a package | `dnf remove httpd` |
| View transaction history | `dnf history` |
| Undo a transaction | `dnf history undo 11` |
| List package groups | `dnf group list` |
| Get group info | `dnf group info "Development Tools"` |
| Install a group | `dnf group install "Development Tools"` |

---

# Errata Types Summary

| Type | Full Name | Purpose | Example ID |
|------|-----------|---------|------------|
| **RHBA** | Red Hat Bugfix Advisory | Bug fixes | `RHBA-2025:9419` |
| **RHEA** | Red Hat Enhancement Advisory | New features/improvements | `RHEA-2025:xxxx` |
| **RHSA** | Red Hat Security Advisory | Security fixes | `RHSA-2025:0960` |

---

# CVE & CVSS Summary

```
CVE-2025-32462
├── Vendor: Neutral (works across Red Hat, Microsoft, Cisco, Debian...)
├── CVSS Score: e.g., 5.5 (Moderate) or 9.8 (Critical)
└── Severity: Low | Moderate | Important | Critical
```

| Severity | CVSS Score Range |
|----------|------------------|
| Low | 0.1 – 3.9 |
| Moderate | 4.0 – 6.9 |
| Important | 7.0 – 8.9 |
| Critical | 9.0 – 10.0 |

---

# DNF vs YUM — Important Note

| Feature | `yum` | `dnf` |
|---------|-------|-------|
| Status | Deprecated | Current |
| Python version | Python 2 (EOL) | Python 3 |
| Command availability | Symlink to `dnf` | Default package manager |

> 💡 The `yum` command still works for **backwards compatibility**, but `dnf` is the **correct** name for RHEL's package manager.

---

# Key Takeaways

- **Registration is required** for updates (use `subscription-manager identity` to verify).
- **BaseOS** = core RHEL components. **AppStream** = applications, runtimes, and tools.
- **DNF** handles automatic dependency resolution — `rpm` does not.
- **Errata** come in three types: Bug fixes (RHBA), Enhancements (RHEA), Security (RHSA).
- **CVE IDs** are vendor-neutral; **CVSS scores** indicate severity.
- One RHSA can address **multiple CVEs**.
- Use `dnf update --security` for security-only updates.
- Use `dnf update --sec-severity=Important --sec-severity=Critical` for highest priority fixes.
- Always check `dnf needs-restarting -r` after updates.
- `dnf history` allows you to **undo** previous transactions.
- **Package groups** install related software together (e.g., "Development Tools").