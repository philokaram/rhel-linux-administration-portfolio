# Managing Server Firewalls — Notes

## 1. Firewall Basics

- Firewalls decide what traffic is **allowed**, **blocked**, or **silently dropped**
- RHEL 10 uses **nftables** under the hood
- **firewalld** is the management interface (you don't manage nftables directly)

### Zones:

- Firewalld organizes rules into **zones** (e.g., `public`, `internal`, `trusted`)
- Each zone has rules for:
  - Services (predefined)
  - Ports
  - Sources
  - Interfaces

> 💡 Instead of memorizing port numbers, enable **predefined services** like `ssh`, `http`, `https`, `cockpit`.

---

## 2. Rule Application Order

```
Incoming Packet
       │
       ▼
1. Check source address
   ├── Source assigned to a zone → Use that zone's rules
   └── Source not assigned → Continue
       │
       ▼
2. Check interface
   ├── Interface assigned to a zone → Use that zone's rules
   └── Interface not assigned → Continue
       │
       ▼
3. Use default zone
```

> 💡 **Order:** Source → Interface → Default zone.

---

## 3. Firewalld Management Tools

| Tool | Description | Recommended For |
|------|-------------|-----------------|
| **Web Console (Cockpit)** | GUI interface | Beginners, quick changes |
| **firewall-cmd** | Command-line tool | Scripting, remote management |

> 💡 **Work smarter:** Use the web console for quick changes, `firewall-cmd` for scripting.

---

## 4. Web Console (Cockpit) — Firewall Management

### Access:

1. Connect to `https://server:9090`
2. Log in (e.g., `student`)
3. Turn on administrative access
4. Navigate to **Networking → Firewall**

### What you can do:

| Action | How |
|--------|-----|
| Enable/disable firewall | Toggle switch |
| View active zones | See current zone(s) |
| Add services | Click **Add services** → select service |
| Add custom ports | Click **Add services** → Add custom ports |
| Create new zone | Click **Add new zone** |

---

## 5. firewall-cmd — Basic Commands

### View zones:

```bash
firewall-cmd --get-zones
```

### View default zone:

```bash
firewall-cmd --get-default-zone
```

### Set default zone:

```bash
firewall-cmd --set-default-zone=trusted
```

### List all rules for a zone:

```bash
firewall-cmd --list-all
```

### List all rules for a specific zone:

```bash
firewall-cmd --zone=trusted --list-all
```

---

## 6. firewall-cmd — Adding Services (Predefined)

### Add service to memory (temporary):

```bash
firewall-cmd --add-service=https
```

### Add service to memory and permanent:

```bash
firewall-cmd --add-service=https --permanent
```

### Add service to memory, then make permanent:

```bash
firewall-cmd --add-service=https
firewall-cmd --runtime-to-permanent
```

### Verify:

```bash
firewall-cmd --list-all
```

> 💡 **Best practice:** Add rules to memory first, test, then convert to permanent with `--runtime-to-permanent`.

---

## 7. firewall-cmd — Adding Custom Ports

### Add port to memory:

```bash
firewall-cmd --add-port=7676/tcp
```

### Make permanent:

```bash
firewall-cmd --runtime-to-permanent
```

### Add with `--permanent` (needs reload):

```bash
firewall-cmd --add-port=7676/tcp --permanent
firewall-cmd --reload
```

---

## 8. firewall-cmd — Adding Sources

### Add a trusted source to a zone:

```bash
firewall-cmd --zone=trusted --add-source=172.25.250.11/32
```

### Make permanent:

```bash
firewall-cmd --runtime-to-permanent
```

---

## 9. firewall-cmd — Reloading Rules

### Reload firewall rules:

```bash
firewall-cmd --reload
```

### Reload behavior:

| Changes | Memory | Persistent |
|---------|--------|------------|
| Before reload | ✅ Active | ✅ Saved |
| After reload | ✅ Active | ✅ Active |

> 💡 **Warning:** Rules added with `--permanent` only (without reload) are not in memory until reload.

---

## 10. Predefined Services

### List all predefined services:

```bash
firewall-cmd --get-services
```

### Service definitions location:

```
/usr/lib/firewalld/services/*.xml   ← System-provided (do not edit)
/etc/firewalld/services/*.xml        ← Custom services (edit here)
```

### Example service definition (Satellite-6.xml):

```xml
<service>
  <short>satellite-6</short>
  <description>Satellite 6</description>
  <include service="foreman"/>
  <port protocol="tcp" port="5000"/>
  <port protocol="tcp" port="5646-5647"/>
</service>
```

---

## 11. Creating a Custom Firewalld Service

### Step 1 — Copy an existing service:

```bash
cp /usr/lib/firewalld/services/http.xml /etc/firewalld/services/listener.xml
```

### Step 2 — Edit the new service:

```bash
vi /etc/firewalld/services/listener.xml
```

**Content:**
```xml
<service>
  <short>listener</short>
  <description>Custom listener service for demonstrations</description>
  <port protocol="tcp" port="14871"/>
</service>
```

### Step 3 — Reload firewalld:

```bash
firewall-cmd --reload
```

### Step 4 — Verify:

```bash
firewall-cmd --get-services | grep listener
```

---

## 12. Runtime vs Permanent — Key Difference

| Command | Effect | Persistence |
|---------|--------|-------------|
| `--add-service` (no flag) | Adds to memory only | Lost on reload/reboot |
| `--add-service --permanent` | Adds to permanent config only | Survives reload/reboot (but not active until reload) |
| `--runtime-to-permanent` | Copies memory rules to permanent | Survives reload/reboot |

### Recommended workflow:

1. Add rules to memory: `firewall-cmd --add-service=https`
2. Test: `curl https://server`
3. If working → make permanent: `firewall-cmd --runtime-to-permanent`

---

## 13. Zone Configuration Files

### System defaults (do not edit):

```
/usr/lib/firewalld/zones/*.xml
```

### Custom zone config (edit here):

```
/etc/firewalld/zones/*.xml
```

### View differences:

```bash
sdiff /usr/lib/firewalld/zones/public.xml /etc/firewalld/zones/public.xml
```

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View all zones | `firewall-cmd --get-zones` |
| View default zone | `firewall-cmd --get-default-zone` |
| Set default zone | `firewall-cmd --set-default-zone=trusted` |
| List all rules (default zone) | `firewall-cmd --list-all` |
| List all rules (specific zone) | `firewall-cmd --zone=trusted --list-all` |
| Add service (memory) | `firewall-cmd --add-service=https` |
| Add service (permanent) | `firewall-cmd --add-service=https --permanent` |
| Add port (memory) | `firewall-cmd --add-port=8080/tcp` |
| Add source to zone | `firewall-cmd --zone=trusted --add-source=IP/32` |
| Make memory rules permanent | `firewall-cmd --runtime-to-permanent` |
| Reload firewall | `firewall-cmd --reload` |
| List predefined services | `firewall-cmd --get-services` |
| View service definition | `cat /etc/firewalld/services/service.xml` |
| View zone configuration | `cat /etc/firewalld/zones/public.xml` |
| View differences | `sdiff /usr/lib/firewalld/zones/public.xml /etc/firewalld/zones/public.xml` |

---

# Firewalld Rule Application — Visual Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    Incoming Packet                          │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
        ┌─────────────────────────┐
        │   Source assigned to    │
        │   a zone?               │
        └────────────┬────────────┘
                     │
         ┌───────────┴───────────┐
         │ Yes                    │ No
         ▼                        ▼
   ┌───────────┐          ┌───────────────┐
   │ Apply     │          │  Interface    │
   │ zone      │          │  assigned to  │
   │ rules     │          │  a zone?      │
   └───────────┘          └───────┬───────┘
                                   │
                     ┌─────────────┴─────────────┐
                     │ Yes                        │ No
                     ▼                            ▼
               ┌───────────┐              ┌─────────────┐
               │ Apply     │              │ Apply       │
               │ zone      │              │ default     │
               │ rules     │              │ zone rules  │
               └───────────┘              └─────────────┘
```

---

# Key Takeaways

- **firewalld** manages firewalls on RHEL 10 (underlying technology: nftables).
- **Zones** organize rules (e.g., `public`, `internal`, `trusted`).
- **Rule order:** Source → Interface → Default zone.
- **Default zone** is `public` (restrictive).
- **Predefined services** eliminate memorizing port numbers.
- **Web Console** is the easiest way to manage firewalld.
- **`firewall-cmd`** is the command-line tool.
- **Runtime vs Permanent:**
  - Runtime rules are lost on reload/reboot.
  - Permanent rules survive reload/reboot.
  - `--runtime-to-permanent` saves runtime rules permanently.
- **Always test** firewall changes before making them permanent.
- **Never edit** `/usr/lib/firewalld/` files — they are owned by RPMs.
- **Custom configurations** go in `/etc/firewalld/` (services, zones).
- **`--reload`** loads permanent configuration into memory.