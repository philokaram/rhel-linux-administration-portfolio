# Editing Network Configuration Files — Notes

## 1. Where NetworkManager Stores Profiles

When you create a profile using `nmcli`, `nmtui`, or the web console, a file is created:

```
/etc/NetworkManager/system-connections/<profile-name>.nmconnection
```

### Example:

```bash
ls -l /etc/NetworkManager/system-connections/
```

```
-rw-------. 1 root root 320 Jan 15 10:00 datacenter.nmconnection
```

---

## 2. File Format — INI Structure

The `.nmconnection` files use the **INI** format (common in Windows).

### Example static configuration:

```ini
[connection]
id=datacenter
uuid=12345678-1234-1234-1234-123456789abc
type=ethernet
interface-name=enp2s0
autoconnect=true

[ethernet]

[ipv4]
method=manual
addresses=192.168.0.10/24
gateway=192.168.0.254
dns=192.168.0.220;1.1.1.1
dns-search=datacenter.example.com;

[ipv6]
method=auto

[proxy]
```

### Section breakdown:

| Section | Purpose |
|---------|---------|
| `[connection]` | Profile metadata (id, uuid, type, interface) |
| `[ethernet]` | Ethernet-specific settings (usually empty) |
| `[ipv4]` | IPv4 configuration (method, addresses, gateway, dns, dns-search) |
| `[ipv6]` | IPv6 configuration |
| `[proxy]` | Proxy settings |

---

## 3. Templating Approach (Student Guide Method)

The student guide suggests:

1. Create a template `.nmconnection` file
2. Make copies for different servers
3. Customize each copy (UUID, IP address, etc.)
4. Distribute to `/etc/NetworkManager/system-connections/` on target servers

### Generate a UUID:

```bash
uuidgen
```

**Output:** `12345678-1234-1234-1234-123456789abc`

---

## 4. Why This Approach Is Error-Prone

| Problem | Consequence |
|---------|-------------|
| No syntax validation | Typos cause profile to fail silently |
| Manual UUID generation | Risk of duplicates |
| Manual IP assignment | Human error (typos, wrong subnet) |
| No dependency management | Interface might not exist on target |
| No atomic updates | Partial config could be applied |

> 💡 **"Boo" — we don't want to do this.**

---

## 5. The Better Way — Ansible

For managing networking at scale, use **Ansible**.

### Ansible Network Role:

- Purpose-built for NetworkManager configuration
- ~10-11 lines in a playbook
- Handles templating, validation, and idempotency

### Example of what Ansible manages:

| Task | Manual | Ansible |
|------|--------|---------|
| Create profile | Edit file by hand | Declarative config |
| Set UUID | Run `uuidgen` manually | Auto-generated |
| Set IP | Type carefully | Variable-driven |
| Distribute | Copy files to each server | Push via playbook |
| Validate | Manual testing | Built-in checks |

> 📌 **Recommendation:** Learn Ansible (RH294 course) for production-scale network management.

---

## 6. When You Might Edit Files Directly

| Scenario | Acceptable? |
|----------|-------------|
| Emergency fix on a single server | Maybe (with caution) |
| Learning/experimentation | Yes (lab environment) |
| Production at scale | **No** (use Ansible) |

---

## 7. If You Must Edit Directly — Checklist

1. ✅ Use `uuidgen` for a new UUID
2. ✅ Verify interface name exists (`ip link show`)
3. ✅ Double-check IP addresses and subnet masks
4. ✅ Ensure gateway is reachable
5. ✅ Set correct file permissions (600 or 644)
6. ✅ Reload NetworkManager: `nmcli con reload`
7. ✅ Activate profile: `nmcli con up <name>`
8. ✅ Validate: `ip a s`, `ip r`, `ping`, `ss -plunt`

---

# Command Reference Table

| Task | Command |
|------|---------|
| Generate UUID | `uuidgen` |
| View profile files | `ls -l /etc/NetworkManager/system-connections/` |
| View file contents | `cat /etc/NetworkManager/system-connections/datacenter.nmconnection` |
| Reload all profiles | `nmcli con reload` |
| Activate profile | `nmcli con up datacenter` |
| Validate IP | `ip a s enp2s0` |
| Validate routing | `ip r` |
| Validate DNS | `cat /etc/resolv.conf` |

---

# INI File Format — Quick Reference

```
[section]
key=value
key2=value2;value3    # Semicolon separated lists
```

### Common keys in `.nmconnection`:

| Section | Key | Example |
|---------|-----|---------|
| `[connection]` | `id` | `datacenter` |
| `[connection]` | `uuid` | `12345678-1234-1234-1234-123456789abc` |
| `[connection]` | `type` | `ethernet` |
| `[connection]` | `interface-name` | `enp2s0` |
| `[connection]` | `autoconnect` | `true` or `false` |
| `[ipv4]` | `method` | `manual` or `auto` (DHCP) |
| `[ipv4]` | `addresses` | `192.168.0.10/24` |
| `[ipv4]` | `gateway` | `192.168.0.254` |
| `[ipv4]` | `dns` | `192.168.0.220;8.8.8.8` |
| `[ipv4]` | `dns-search` | `example.com;lab.example.com` |

---

# Manual vs Ansible — Comparison

| Aspect | Manual (Edit Files) | Ansible Network Role |
|--------|---------------------|----------------------|
| Syntax validation | ❌ None | ✅ Built-in |
| UUID generation | Manual (`uuidgen`) | Automatic |
| Scalability | Low (per-server) | High (push to many) |
| Error risk | High | Low |
| Idempotency | ❌ No | ✅ Yes |
| Audit trail | None (file changes) | Full (playbook history) |
| Learning curve | Low | Moderate |
| Production recommendation | ❌ Avoid | ✅ Use |

---

# Key Takeaways

- **Profile files** live in `/etc/NetworkManager/system-connections/*.nmconnection`
- **INI format** — sections in `[brackets]`, keys with values
- **Templating approach** is possible but **error-prone** (no syntax validation)
- **`uuidgen`** generates unique identifiers for profiles
- **Ansible is the better solution** for managing networking at scale (RH294 course)
- **Manual editing** = emergency/lab only, not production at scale
- **Work smarter, not harder** — don't manually edit network config files when automation tools exist
- If you must edit directly: validate, reload, test, and be prepared to fix mistakes