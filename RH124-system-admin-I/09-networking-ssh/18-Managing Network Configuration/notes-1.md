# Configuring Networking from the Command Line — Notes

## 1. NetworkManager — The Heart of RHEL Networking

- **NetworkManager** = service responsible for managing networking on RHEL (introduced in RHEL 7)
- Based on **profiles** (also called connections)
- Each profile supplies settings to **one interface**
- An interface can only receive settings from **one profile** at a time

### Profile structure:

```
Profile → contains properties.attributes → values
Example: ipv4.addresses = 192.168.0.10/24
```

---

## 2. Tools to Manage NetworkManager

| Tool | Description | Best for |
|------|-------------|----------|
| **nmcli** | Command-line tool | Scripting, remote management |
| **nmtui** | Text-based UI (curses) | **Tricky environments**, quick config |
| **Cockpit** | Web console | GUI lovers, remote management |
| **Ansible** | Automation | Infrastructure as code |
| **Editing files directly** | `/etc/NetworkManager/system-connections/*.nmconnection` | **Not recommended** (error-prone) |

> 💡 **Recommendation:** Use `nmcli` for scripting, `nmtui` for interactive/emergency config. Avoid editing files directly — no syntax validation.

---

## 3. `nmcli` — Command-Line Basics

### View all connections (profiles):

```bash
nmcli
```

or

```bash
nmcli connection show
```

Short form:

```bash
nmcli con show
```

### What the output shows:

| Column | Meaning |
|--------|---------|
| NAME | Profile name |
| UUID | Unique identifier |
| TYPE | Connection type (ethernet, wifi, bond, etc.) |
| DEVICE | Interface associated with profile |
| Color | Green = active (supplying settings), White = inactive |

> 💡 Use **tab completion** extensively with `nmcli` — press `Tab` twice to see available subcommands.

---

## 4. Creating a New Profile — `nmcli con add`

### Basic syntax:

```bash
nmcli con add con-name <name> type <type> ifname <interface>
```

### Example — create static profile for `enp2s0`:

```bash
nmcli con add con-name datacenter type ethernet ifname enp2s0 \
  ipv4.addresses 192.168.0.10/24 \
  ipv4.gateway 192.168.0.254 \
  ipv4.dns 192.168.0.220 \
  ipv4.method manual
```

| Parameter | Meaning |
|-----------|---------|
| `con-name` | Profile name (e.g., `datacenter`) |
| `type` | Connection type (`ethernet`, `wifi`, `bond`) |
| `ifname` | Interface name (e.g., `enp2s0`) |
| `ipv4.method manual` | Static configuration (default = `auto` / DHCP) |

> 💡 `ipv4.addresses` is plural because it accepts an **array** of IP addresses (comma-separated).

---

## 5. Viewing Profile Details — `nmcli con show`

### Show all profile settings:

```bash
nmcli con show datacenter
```

### Filter by property (show only IPv4 settings):

```bash
nmcli con show datacenter | grep ipv4
```

### Show specific attributes (comma-separated):

```bash
nmcli con show datacenter --fields ipv4.dns,ipv4.dns-search,ipv4.addresses
```

---

## 6. Modifying Profiles — `nmcli con mod`

### Operators:

| Operator | Effect |
|----------|--------|
| (none) | **Overwrite** existing value |
| `+` | **Add** to existing value |
| `-` | **Remove** from existing value |

### Examples:

**Overwrite DNS server (replaces existing):**

```bash
nmcli con mod datacenter ipv4.dns 1.1.1.1
```

**Add a DNS server (keeps existing):**

```bash
nmcli con mod datacenter +ipv4.dns 8.8.8.8
```

**Add an additional IP address:**

```bash
nmcli con mod datacenter +ipv4.addresses 10.10.10.10/24
```

**Remove an IP address:**

```bash
nmcli con mod datacenter -ipv4.addresses 10.10.10.10/24
```

**Add a DNS search domain:**

```bash
nmcli con mod datacenter +ipv4.dns-search classroom.example.com
```

**Set (overwrite) DNS search domain:**

```bash
nmcli con mod datacenter ipv4.dns-search datacenter.example.com
```

---

## 7. Applying Changes — `nmcli con up` or `nmcli con reload`

### Reactivate a profile (applies changes immediately):

```bash
nmcli con up datacenter
```

### Reload all profiles:

```bash
nmcli con reload
```

> 💡 After modifying a profile, you must **reactivate** it or reload for changes to take effect.

---

## 8. Deleting a Profile — `nmcli con del`

```bash
nmcli con del "Wired connection 1"
```

> 💡 Use tab completion for names with spaces — it will escape them automatically.

---

## 9. `nmtui` — The Emergency Tool

If `nmcli` syntax feels overwhelming in a tricky environment, use **`nmtui`**:

```bash
nmtui
```

### Interactive workflow:

1. **Edit a connection** → Choose profile
2. **Add** → Ethernet → Name + Interface
3. **IPv4 CONFIGURATION** → Change to **Manual**
4. **Show** → Add IP address, Gateway, DNS servers, Search domains
5. **OK** → **Back** → **Quit**

> 💡 `nmtui` provides syntax validation and is much faster than memorizing `nmcli` syntax. Less than 1 minute to configure a profile.

---

## 10. Validation After Configuration

### IP address:

```bash
ip address show enp2s0
```

### Routing table:

```bash
ip route show
```

### DNS configuration:

```bash
cat /etc/resolv.conf
```

**Example output:**
```
search lab.example.com example.com datacenter.example.com
nameserver 192.168.0.220
nameserver 1.1.1.1
```

> 📌 The resolver may not use more than **3 nameservers** or **3 search domains**.

---

## 11. Profile Storage Location

All profiles are stored as `.nmconnection` files:

```bash
ls -l /etc/NetworkManager/system-connections/
```

**Example:**
```
-rw-------. 1 root root 320 Jan 15 10:00 datacenter.nmconnection
-rw-------. 1 root root 280 Jan 14 09:00 cloud-init-ens3.nmconnection
```

### Why you shouldn't edit these directly:

- No syntax validation
- Error-prone
- Must restart NetworkManager or reload config

> 💡 Use `nmcli` or `nmtui` — they provide syntax validation.

---

## 12. Privileges

- Only **root** can manage NetworkManager via CLI
- Exception: Users logged into **physical or virtual console** (e.g., laptop) can manage connections (wireless, etc.)

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| List all connections | `nmcli con show` |
| Show profile details | `nmcli con show datacenter` |
| Show filtered profile details | `nmcli con show datacenter \| grep ipv4` |
| Show specific fields | `nmcli con show datacenter --fields ipv4.dns,ipv4.addresses` |
| Add static profile | `nmcli con add con-name datacenter type ethernet ifname enp2s0 ipv4.method manual ipv4.addresses 192.168.0.10/24 ipv4.gateway 192.168.0.254 ipv4.dns 192.168.0.220` |
| Modify profile (overwrite) | `nmcli con mod datacenter ipv4.dns 1.1.1.1` |
| Modify profile (add) | `nmcli con mod datacenter +ipv4.dns 8.8.8.8` |
| Modify profile (remove) | `nmcli con mod datacenter -ipv4.addresses 10.10.10.10/24` |
| Apply changes (reactivate) | `nmcli con up datacenter` |
| Reload all profiles | `nmcli con reload` |
| Delete profile | `nmcli con del "Wired connection 1"` |
| Interactive text UI | `nmtui` |
| View profile files | `ls -l /etc/NetworkManager/system-connections/` |
| Validate IP config | `ip a s enp2s0` |
| Validate routing | `ip r` |
| Validate DNS | `cat /etc/resolv.conf` |

---

# `nmcli` Operator Cheat Sheet

| You want to... | Operator | Example |
|----------------|----------|---------|
| Replace value | (none) | `nmcli con mod profile ipv4.dns 1.1.1.1` |
| Add to existing | `+` | `nmcli con mod profile +ipv4.dns 8.8.8.8` |
| Remove from existing | `-` | `nmcli con mod profile -ipv4.addresses 10.0.0.1/24` |

---

# `nmcli` Subcommand Abbreviation Reference

| Full Command | Abbreviation |
|--------------|--------------|
| `nmcli connection` | `nmcli con` |
| `nmcli connection show` | `nmcli con show` |
| `nmcli connection add` | `nmcli con add` |
| `nmcli connection modify` | `nmcli con mod` |
| `nmcli connection up` | `nmcli con up` |
| `nmcli connection down` | `nmcli con down` |
| `nmcli connection delete` | `nmcli con del` |
| `nmcli connection reload` | `nmcli con reload` |

---

# Profile Settings — Key Properties

| Property | Attribute | Example Value | Meaning |
|----------|-----------|---------------|---------|
| `connection` | `id` | `datacenter` | Profile name |
| `connection` | `interface-name` | `enp2s0` | Associated interface |
| `ipv4` | `method` | `manual` or `auto` | Static or DHCP |
| `ipv4` | `addresses` | `192.168.0.10/24` | IP + subnet mask |
| `ipv4` | `gateway` | `192.168.0.254` | Default gateway |
| `ipv4` | `dns` | `192.168.0.220` | DNS server |
| `ipv4` | `dns-search` | `example.com` | DNS search domain |

---

# Key Takeaways

- **NetworkManager** = central service for networking on RHEL (since RHEL 7).
- **Profiles** (connections) supply settings to interfaces (one interface → one profile).
- **`nmcli`** = powerful command-line tool with tab completion (use it).
- **`nmtui`** = life-saver in tricky environments — simple, fast, less error-prone.
- **Never edit** `/etc/NetworkManager/system-connections/*.nmconnection` directly — error-prone, no syntax validation.
- **Operators** in `nmcli con mod`: (none)=overwrite, `+`=add, `-`=remove.
- After modifications, use `nmcli con up <profile>` or `nmcli con reload` to apply changes.
- Validate with `ip a s`, `ip r`, `cat /etc/resolv.conf`.
- The resolver may not honor more than **3 nameservers** or **3 search domains**.
- Only **root** can manage NetworkManager (except console logins for desktop use).
- **Work smarter, not harder** — use `nmtui` when `nmcli` syntax is overwhelming.