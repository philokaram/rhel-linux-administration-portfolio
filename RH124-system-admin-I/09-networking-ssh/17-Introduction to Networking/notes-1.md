# Networking Concepts — Notes

## 1. Why Networking Matters

- Networking = how Linux systems talk to each other and the outside world
- To manage Linux effectively, you need to understand network basics

---

## 2. The TCP/IP Model — Four Layers

Linux cares most about the TCP/IP model:

| Layer | Purpose | Examples |
|-------|---------|----------|
| **Link** | Physical or virtual connection | Ethernet, Wi-Fi |
| **Internet** | IP addresses and routing | IPv4, IPv6 |
| **Transport** | Protocols for data transfer | TCP, UDP |
| **Application** | Services that use the network | HTTP, SSH, DNS |

---

## 3. Network Interfaces — Connection Points

A **network interface** is the connection point between your system and the network.

### Interface naming:

| Type | Name Example | Meaning |
|------|--------------|---------|
| Legacy | `eth0`, `wlan0` | Old naming scheme |
| Modern (predictable) | `enp1s0` | `en`=ethernet, `p1`=port 1, `s0`=slot 0 |

> 💡 Predictable names ensure that if a network card fails, replacing it in the same physical slot keeps the same interface name — no configuration loss.

### Special interface — LOOPBACK (`lo`):

- Virtual interface
- Always `UP`
- Used for local communication (127.0.0.1)

---

## 4. Viewing Network Interfaces

### Show IP addresses (all interfaces):

```bash
ip address show
```

Short form:

```bash
ip a s
```

### Show specific interface:

```bash
ip a s enp1s0
```

### Show link status (MAC addresses, UP/DOWN):

```bash
ip link show
```

Short form:

```bash
ip l s
```

**Shows:**
- Interface state (UP/DOWN)
- MAC address (link/ether)

---

## 5. IP Addresses — IPv4 and IPv6

### IPv4:

- **32 bits** long
- Written as **4 decimal numbers** (octets)
- Example: `172.25.250.10`

### IPv6:

- **128 bits** long
- Written in **hexadecimal with colons** (`:`)
- Example: `2001:db8::1`

### Why IPv6?

- Running out of IPv4 addresses
- Vastly larger address space
- Built-in improvements: routing, auto-configuration (SLAAC)

---

## 6. IP Address Structure — Network + Host

An IP address has two parts:

| Part | Purpose |
|------|---------|
| **Network ID** | Identifies the network |
| **Host ID** | Identifies the specific device on that network |

### Subnet mask / CIDR prefix:

Tells you where the split happens.

### Example:

```
192.168.5.3/24
```

| Component | Value |
|-----------|-------|
| IP address | 192.168.5.3 |
| /24 prefix | First 24 bits = network |
| Network ID | 192.168.5 |
| Host ID | 3 |

---

## 7. Subnet Mask — Binary Math (ANDing)

### IPv4 octet values (bits):

| Bit position | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 |
|--------------|---|---|---|---|---|---|---|---|
| Value | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

- All bits enabled = 255 (128+64+32+16+8+4+2+1)
- `2^8 = 256` combinations (0-255)

### ANDing logic:

| AND | Result |
|-----|--------|
| 1 AND 1 | 1 |
| 1 AND 0 | 0 |
| 0 AND 1 | 0 |

> 💡 ANDing an IP address with its subnet mask reveals the **network portion**.

### Example — determine if routing is needed:

Two IPs on different networks require a **router** to communicate.

---

## 8. Private IP Address Ranges (Not Routable on Internet)

| Range | CIDR | Subnet Mask | Use |
|-------|------|-------------|-----|
| `10.0.0.0` – `10.255.255.255` | /8 | 255.0.0.0 | Large networks |
| `172.16.0.0` – `172.31.255.255` | /12 | 255.240.0.0 | Medium networks |
| `192.168.0.0` – `192.168.255.255` | /16 | 255.255.0.0 | Small networks |

### Link-local range (automatic fallback):

| Range | CIDR | When used |
|-------|------|-----------|
| `169.254.0.0` – `169.254.255.255` | /16 | No other IP assigned (DHCP fails) |

---

## 9. Calculating IP Ranges — `ipcalc`

Instead of manual binary math, use **`ipcalc`**.

### Install:

```bash
dnf install -y ipcalc
```

### Basic usage:

```bash
ipcalc 172.25.250.0/24
```

**Shows:**
- Network address
- Netmask (decimal)
- Broadcast address
- Host range (first to last)
- Number of hosts
- Address type (private/public)

### Variable-length subnet mask (VLSM) example:

```bash
ipcalc 172.25.18.0/22
```

**Output:**
- Network: `172.25.16.0/22`
- Netmask: `255.255.252.0`
- Broadcast: `172.25.19.255`
- Host range: `172.25.16.1` – `172.25.19.254`
- Hosts: 1022
- Private use: Yes

> 💡 `/22` means 22 bits for network, 10 bits for hosts (`2^10 - 2 = 1022` usable hosts).

---

## 10. IP Assignment Methods

| Method | How it works | Use case |
|--------|--------------|----------|
| **Static** | Manually assigned via config files | Servers, infrastructure |
| **Dynamic (DHCPv4)** | DHCP server assigns IP | Clients, workstations |

### IPv6 assignment:

| Method | Description |
|--------|-------------|
| **DHCPv6** | Similar to DHCPv4 — server assigns addresses |
| **SLAAC** | Stateless Address Autoconfiguration — system generates its own IP based on router announcements |

> 💡 SLAAC is one reason IPv6 is easier to manage at scale.

---

## 11. Gateways — Routing Traffic

- **Gateway** = router that forwards traffic outside your local network
- To reach anything outside your network, traffic goes through the default gateway

### View routing table:

```bash
ip route show
```

Short forms:

```bash
ip r s
ip r
```

### Example output:

```
default via 172.25.250.254 dev enp1s0
```

| Field | Meaning |
|-------|---------|
| `default` | Default route (0.0.0.0/0) |
| `via 172.25.250.254` | Gateway IP address |
| `dev enp1s0` | Network interface used |

---

## 12. DNS — Translating Names to IPs

- Humans prefer names (`www.example.com`)
- DNS translates names to IP addresses

### DNS lookup tools:

| Command | Example | Pros/Cons |
|---------|---------|-----------|
| `getent hosts` | `getent hosts www.redhat.com` | Uses system's name resolution (files + DNS) |
| `dig` | `dig www.redhat.com` | Detailed output, shows ANSWER SECTION |
| `host` | `host www.redhat.com` | Simple, shows aliases (CNAMEs) and IPs |

### Example `dig` output:

```
;; ANSWER SECTION:
www.redhat.com.    300    IN    A    172.25.250.10
www.redhat.com.    300    IN    AAAA    2001:db8::1
```

- `A` record = IPv4 address
- `AAAA` record = IPv6 address

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Show IP addresses (all interfaces) | `ip a s` or `ip address show` |
| Show specific interface | `ip a s enp1s0` |
| Show link status + MAC addresses | `ip l s` or `ip link show` |
| Show routing table | `ip r` or `ip route show` |
| Install ipcalc | `dnf install -y ipcalc` |
| Calculate network range | `ipcalc 172.25.18.0/22` |
| DNS lookup (system resolution) | `getent hosts www.redhat.com` |
| DNS lookup (detailed) | `dig www.redhat.com` |
| DNS lookup (simple) | `host www.redhat.com` |

---

# TCP/IP Model — Quick Reference

```
Application Layer    ← HTTP, SSH, DNS (your services)
       ↑
Transport Layer      ← TCP (reliable), UDP (fast)
       ↑
Internet Layer       ← IP addressing, routing (IPv4, IPv6)
       ↑
Link Layer           ← Ethernet, Wi-Fi (physical/virtual connection)
```

---

# Private IP Ranges Cheat Sheet

| Range | CIDR | Number of Hosts | Typical Use |
|-------|------|-----------------|-------------|
| `10.0.0.0 – 10.255.255.255` | /8 | 16,777,214 | Large enterprises |
| `172.16.0.0 – 172.31.255.255` | /12 | 1,048,574 | Medium organizations |
| `192.168.0.0 – 192.168.255.255` | /16 | 65,534 | Home/Small office |
| `169.254.0.0 – 169.254.255.255` | /16 | 65,534 | Link-local (fallback) |

---

# Subnet Mask CIDR Reference

| CIDR | Subnet Mask | Number of Hosts |
|------|-------------|-----------------|
| /8 | 255.0.0.0 | 16,777,214 |
| /16 | 255.255.0.0 | 65,534 |
| /22 | 255.255.252.0 | 1,022 |
| /24 | 255.255.255.0 | 254 |
| /30 | 255.255.255.252 | 2 (point-to-point) |

> Formula: `2^(32 - CIDR) - 2` = usable hosts

---

# Key Takeaways

- **TCP/IP model** = Link, Internet, Transport, Application layers.
- **`ip` command** = modern replacement for `ifconfig` and `route` (use `ip`, not `ifconfig`).
- **Network interfaces** have predictable names (`enp1s0` = ethernet, port 1, slot 0).
- **`lo`** = loopback interface (127.0.0.1) — always UP.
- **IPv4** = 32 bits, 4 octets. **IPv6** = 128 bits, hexadecimal with colons.
- **Subnet mask** tells you network portion vs host portion.
- **Private IP ranges** (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) are not routable on the internet.
- **`ipcalc`** calculates network ranges — much easier than binary math.
- **Default gateway** = router for external traffic (`ip route show`).
- **DNS** translates names to IPs — use `getent hosts`, `dig`, or `host`.
- **IPv6 advantages:** Larger address space, built-in auto-configuration (SLAAC).