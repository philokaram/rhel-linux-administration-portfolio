# Automating RHEL Installation with Kickstart — Notes

## 1. What Is Kickstart?

- **Kickstart** = a plaintext file that provides answers to Anaconda's questions
- Enables **non-interactive**, automated installation of RHEL
- Great for deploying many systems consistently

### Key benefits:

| Benefit | Description |
|---------|-------------|
| **Repeatability** | Same installation every time |
| **Scalability** | Deploy hundreds of systems |
| **Consistency** | Same packages, settings, partitions |
| **Automation** | No manual intervention needed |

---

## 2. Kickstart Generator

### URL:

```
access.redhat.com/labs/kickstartconfig
```

### What it provides:

- Good foundation for a Kickstart file
- Not 100% comprehensive, but a great starting point

### Options in the generator:

| Section | Options |
|---------|---------|
| **General** | Language, keyboard, timezone, text/graphical mode |
| **Installation Source** | HTTP, FTP, NFS, local media |
| **Partitioning** | Automatic or manual |
| **Root Password** | Set root password |
| **GRUB Password** | Secure the boot menu |
| **Kernel Parameters** | RDP options, console settings |
| **Package Selection** | Package groups and individual packages |
| **Pre/Post Scripts** | Commands before/after installation |

---

## 3. Kickstart File — Key Sections

### Section 1 — System Settings

```
lang en_US
keyboard us
timezone UTC --isUtc
rootpw --lock
text
reboot
```

| Directive | Meaning |
|-----------|---------|
| `lang` | Language |
| `keyboard` | Keyboard layout |
| `timezone` | Time zone |
| `rootpw --lock` | Lock root account (no password login) |
| `text` | Text mode installation (no GUI) |
| `reboot` | Reboot after installation |

---

### Section 2 — User Creation

```
user --name=ricardodacosta --password=redhat123 --groups=wheel
```

| Option | Meaning |
|--------|---------|
| `--name` | Username |
| `--password` | User password |
| `--groups` | Groups to add user to |

> 💡 Adding user to `wheel` group gives them `sudo` access.

---

### Section 3 — Installation Source

```
url --url="http://content.example.com/dvd/"
```

| Option | Meaning |
|--------|---------|
| `url` | HTTP/HTTPS/FTP location of installation media |
| `--url` | The actual URL |

---

### Section 4 — Partitioning

```
clearpart --all --initlabel
autopart
```

| Directive | Meaning |
|-----------|---------|
| `clearpart --all` | Remove all existing partitions |
| `--initlabel` | Initialize disk label |
| `autopart` | Use automatic partitioning |

---

### Section 5 — Additional Repositories

```
repo --name="AppStream" --baseurl="http://content.example.com/dvd/AppStream"
```

| Option | Meaning |
|--------|---------|
| `repo` | Add a repository |
| `--name` | Repository name |
| `--baseurl` | Repository URL |

---

### Section 6 — Bootloader and GRUB

```
bootloader --password=secret --timeout=5
```

| Option | Meaning |
|--------|---------|
| `--password` | GRUB password (secure boot menu) |
| `--timeout` | GRUB timeout in seconds |

### Kernel parameters (RDP):

```
%kernel
inst.rdp
inst.rdp.username=ricardodacosta
inst.rdp.password=redhat123
%end
```

---

### Section 7 — Firewall

```
firewall --enabled --service=ssh --service=rdp
```

| Option | Meaning |
|--------|---------|
| `--enabled` | Enable firewall |
| `--service` | Services to allow (ssh, rdp) |

> 💡 Allow `rdp` service if using RDP installation.

---

### Section 8 — SELinux

```
selinux --enforcing
```

| Option | Meaning |
|--------|---------|
| `--enforcing` | Enforce SELinux (recommended) |
| `--permissive` | Permissive mode (troubleshooting) |
| `--disabled` | Disable SELinux (not recommended) |

---

### Section 9 — Network

```
network --device=link --bootproto=dhcp
```

| Option | Meaning |
|--------|---------|
| `--device` | Network device |
| `--bootproto` | DHCP or static |

---

### Section 10 — Packages

```
%packages
@^server-product-environment
@development
podman
buildah
vim*
%end
```

| Syntax | Meaning |
|--------|---------|
| `@^` | Environment group (with caret) |
| `@` | Package group |
| `package-name` | Individual package |
| `*` | Wildcard (all matching packages) |

---

### Section 11 — Pre-Installation Script

```
%pre
#!/bin/bash
# Commands to run before installation starts
%end
```

### Section 12 — Post-Installation Script

```
%post
#!/bin/bash
# Commands to run after installation completes

# Update man database
mandb

# Remove default MOTD files
rm -f /etc/motd.d/*

# Create custom MOTD
echo "RHEL10 installed with Kickstart" > /etc/motd.d/kickstart
%end
```

---

## 4. Kickstart Validation — KS Validator

### Install validator:

```bash
dnf install -y pykickstart
```

### Validate Kickstart file:

```bash
ksvalidator kickstart.txt
```

**Expected:** No output = valid file.

> 💡 Always validate your Kickstart file before deployment.

---

## 5. Deploying Kickstart — PXE Boot

### Add Kickstart to boot command:

```
inst.ks=http://servera/kickstart.txt
```

### Example PXE boot:

```
linux vmlinuz inst.ks=http://servera/kickstart.txt
```

---

## 6. Image Builder — Alternative to Kickstart

### URL:

```
console.redhat.com/insights → Inventory → Images
```

### What it does:

- Create custom **image blueprints**
- Build custom installation ISOs
- Pre-configure packages, partitioning, security profiles

### Benefits:

| Benefit | Description |
|---------|-------------|
| **Custom images** | Build once, deploy many |
| **No installation** | Deploy pre-configured images |
| **Hosted service** | Available at console.redhat.com |

---

## 7. Kickstart Post-Install — MOTD Example

### Custom MOTD file:

```
/etc/motd.d/kickstart
```

**Content:**
```
RHEL10 is installed with the Kickstart file supplying answers to Anaconda
```

> 💡 MOTD files in `/etc/motd.d/` are displayed at login.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Install KS validator | `dnf install -y pykickstart` |
| Validate Kickstart file | `ksvalidator kickstart.txt` |
| Copy to web server | `rsync -av kickstart.txt servera:/var/www/html/` |
| Add Kickstart to boot | `inst.ks=http://servera/kickstart.txt` |

---

# Kickstart File — Complete Example

```
lang en_US
keyboard us
timezone UTC --isUtc
rootpw --lock
user --name=ricardodacosta --password=redhat123 --groups=wheel
text
reboot
url --url="http://content.example.com/dvd/"
repo --name="AppStream" --baseurl="http://content.example.com/dvd/AppStream"
clearpart --all --initlabel
autopart
bootloader --password=secret --timeout=5
selinux --enforcing
firewall --enabled --service=ssh --service=rdp
network --device=link --bootproto=dhcp

%kernel
inst.rdp
inst.rdp.username=ricardodacosta
inst.rdp.password=redhat123
%end

%packages
@^server-product-environment
@development
podman
buildah
vim*
%end

%post --interpreter=/bin/bash
mandb
rm -f /etc/motd.d/*
echo "RHEL10 installed with Kickstart" > /etc/motd.d/kickstart
%end
```

---

# Key Takeaways

- **Kickstart** = automated, non-interactive RHEL installation.
- Uses a **plaintext file** to answer Anaconda's questions.
- **Kickstart Generator** provides a solid starting point.
- **KS Validator** (`ksvalidator`) checks for syntax errors.
- **Secure GRUB** with `bootloader --password`.
- **Lock root account** (`rootpw --lock`) and create a **user with sudo**.
- **RDP kernel parameters** (`inst.rdp`, `inst.rdp.username`, `inst.rdp.password`).
- **Post-install scripts** can run commands after installation (e.g., custom MOTD).
- **Image Builder** is an alternative to Kickstart for custom ISOs.
- **Always test** Kickstart files in a lab before production deployment.
- **PXE boot** + Kickstart = large-scale automated deployments.