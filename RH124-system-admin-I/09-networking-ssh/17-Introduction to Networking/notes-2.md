# Validating Network Configuration — Notes

## 1. The One Command — `ping`

If you ask me to validate that networking is working, I'm going to use `ping`.

```bash
ping -c1 redhat.com
```

| What it tests | Why it matters |
|---------------|----------------|
| Valid IP address | System has proper IP config |
| Connectivity to remote host | Routing + gateway work |
| DNS resolution | Name translation works |

> 💡 One command tests three critical things.

---

## 2. Viewing IP Configuration — `ip address show`

### All interfaces:

```bash
ip address show
```

Short form:

```bash
ip a s
```

### Specific interface:

```bash
ip address show enp1s0
```

### What it shows:

| Field | Meaning |
|-------|---------|
| `link/ether` | MAC address |
| `inet` | IPv4 address + subnet mask (CIDR) |
| `brd` | Broadcast address |
| `inet6` | IPv6 address |
| `scope link` | Link-local address (only on this network segment) |

> 💡 IPv6 link-local addresses are automatically assigned and only valid on the local network segment.

---

## 3. Testing IPv6 Connectivity — `ping6`

```bash
ping6 fe80::1234:5678:abcd:ef01%enp1s0
```

| Part | Meaning |
|------|---------|
| IPv6 address | Link-local address of target |
| `%enp1s0` | **Interface** (required for link-local addresses) |

> ⚠️ Link-local addresses require the interface name because the same address could exist on multiple interfaces.

```bash
ping6 -c4 fe80::1234:5678:abcd:ef01%enp1s0
```

Press `Ctrl+C` to stop (ping6 runs continuously by default).

---

## 4. Routing Table — `ip route`

### IPv4 routing table:

```bash
ip route show
```

Short form:

```bash
ip r
```

### IPv6 routing table:

```bash
ip -6 route show
```

Short form:

```bash
ip -6 r
```

### Example output:

```
default via 172.25.250.254 dev enp1s0
```

| Field | Meaning |
|-------|---------|
| `default` | Default route (0.0.0.0/0) |
| `via 172.25.250.254` | Gateway IP address |
| `dev enp1s0` | Interface used |

---

## 5. DNS Configuration — `/etc/resolv.conf`

```bash
cat /etc/resolv.conf
```

**Example output:**

```
search lab.example.com example.com
nameserver 10.11.5.19
nameserver 172.25.250.220
```

| Directive | Meaning |
|-----------|---------|
| `search` | Domain suffixes to try for short hostnames |
| `nameserver` | DNS server IP addresses (queried in order) |

### How search works:

Query `ping www` (short name):

1. Try `www.lab.example.com` → if fails
2. Try `www.example.com` → if fails
3. Resolution fails

---

## 6. Ultimate Troubleshooting Tool — `mtr`

`mtr` = My Traceroute (combines `ping` + `traceroute`)

### Interactive mode:

```bash
mtr www.redhat.com
```

### What it shows:

- Every router (hop) between you and the destination
- Running statistics: packet loss, latency
- Updates in real time

### Key columns:

| Column | Meaning |
|--------|---------|
| Loss% | Packet loss percentage |
| Snt | Packets sent |
| Last | Latency of last packet |
| Avg | Average latency |
| Best | Best (shortest) latency |
| Wrst | Worst (longest) latency |
| StDev | Standard deviation (jitter indicator) |

> 💡 If a service is slow, run `mtr` for 500-1000 packets to get meaningful data.

### Report mode (non-interactive):

```bash
mtr --report -c 1000 www.redhat.com
```

| Option | Meaning |
|--------|---------|
| `--report` | Generate one-time report (exit after) |
| `-c 1000` | Send 1000 packets |
| `-n` | Show IPs (not hostnames) |
| `-4` | IPv4 only |
| `-6` | IPv6 only |

### Quit interactive mode:

Press `q`

---

## 7. Alternative Tracing Tools (Less Recommended)

| Command | Pros | Cons |
|---------|------|------|
| `tracepath` | No root needed | One-off latencies, no running stats |
| `traceroute` | More detail than tracepath | May need installation, one-off |

### tracepath:

```bash
tracepath www.redhat.com
```

Press `Ctrl+C` to stop.

### traceroute (install if needed):

```bash
dnf install -y traceroute
traceroute www.redhat.com
```

> 💡 **Recommendation:** Use `mtr` — it's the ultimate tool.

---

## 8. Viewing Open Ports and Processes — `ss`

### The magic command:

```bash
ss -plunt
```

### Option breakdown (mnemonic: **plunt**):

| Option | Meaning |
|--------|---------|
| `-p` | Show **p**rocesses |
| `-l` | Show **l**istening sockets |
| `-u` | Show **u**dp |
| `-n` | Show **n**umbers (no service names, e.g., 22 not ssh) |
| `-t` | Show **t**cp |

> 💡 Order doesn't matter — `ss -plunt`, `ss -tulpn`, `ss -nltup` all work.

### Filter for specific port:

```bash
ss -plunt | grep :22
```

**Example output:**
```
LISTEN 0      128   0.0.0.0:22   0.0.0.0:*    users:(("sshd",pid=991,fd=3))
LISTEN 0      128      [::]:22      [::]:*    users:(("sshd",pid=991,fd=4))
```

### Understanding the output:

| Field | Meaning |
|-------|---------|
| `0.0.0.0:22` | Listening on all IPv4 interfaces, port 22 |
| `[::]:22` | Listening on all IPv6 interfaces, port 22 |
| `users:(("sshd",pid=991))` | Process name and PID |

> 💡 `0.0.0.0` = all IPv4 interfaces. `::` = all IPv6 interfaces (in square brackets).

### Common ports to check:

| Port | Service |
|------|---------|
| 22 | SSH (sshd) |
| 80 | HTTP (httpd) |
| 443 | HTTPS (httpd) |
| 9090 | Cockpit (web console) |

---

## 9. Legacy Commands — `netstat`

```bash
netstat -plunt
```

- Older command, still widely used
- May need installation (not always available on minimal installs)
- `ss` is the modern replacement

> 💡 `ss` is guaranteed on even minimal RHEL installations.

---

## 10. Link Status and Statistics — `ip link`

### Show interface status:

```bash
ip link show
```

or

```bash
ip l s
```

**Shows:** Interface name, state (UP/DOWN), MAC address

### Show statistics (packet counts):

```bash
ip -s link show enp1s0
```

**Shows:**
- RX (received) packets, bytes, errors, dropped
- TX (transmitted) packets, bytes, errors, dropped

> 💡 Same info is available from `ip address show` plus more.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Quick connectivity test | `ping -c1 redhat.com` |
| View IP configuration | `ip a s` or `ip address show` |
| View specific interface | `ip a s enp1s0` |
| Test IPv6 (link-local) | `ping6 fe80::...%enp1s0` |
| View IPv4 routing | `ip r` or `ip route show` |
| View IPv6 routing | `ip -6 r` |
| View DNS config | `cat /etc/resolv.conf` |
| Interactive network diagnostics | `mtr redhat.com` |
| Report mode (1000 packets) | `mtr --report -c 1000 redhat.com` |
| View open ports + processes | `ss -plunt` |
| Filter by port | `ss -plunt \| grep :22` |
| Legacy port viewer | `netstat -plunt` |
| View link status | `ip l s` |
| View interface statistics | `ip -s link show enp1s0` |

---

# Troubleshooting Workflow

```
1. ping -c1 redhat.com
   ├── FAILS → Check DNS (/etc/resolv.conf)
   ├── FAILS → Check routing (ip r)
   └── FAILS → Check interface (ip a s)

2. mtr redhat.com
   ├── Packet loss at specific hop → Issue with that router
   ├── High latency → Network congestion or distance
   └── Clean output → Network is healthy

3. ss -plunt
   ├── Service not listening → Check service status
   ├── Service listening on wrong IP → Check config
   └── Wrong port → Check config

4. ip -s link show enp1s0
   ├── RX errors → Physical/Cable issue
   ├── TX errors → Physical/Cable issue
   └── Dropped packets → Congestion or buffer issues
```

---

# `mtr` Output Interpretation

```
Start: 2025-01-15T10:00:00
HOST: servera                    Loss%   Snt   Last   Avg  Best  Wrst StDev
  1. gateway.example.com         0.0%    27    0.3   0.5   0.3   1.1   0.2
  2. router1.example.com         0.0%    27    1.2   1.4   1.1   2.3   0.3
  3. core-router.example.com     5.0%    27   15.2  16.1  14.8  18.2   1.1
  4. www.redhat.com              0.0%    27   14.9  15.8  14.5  17.9   1.0
```

| Red Flag | Meaning |
|----------|---------|
| Loss% > 0 at hop 3, clears at hop 4 | Router is rate-limiting ICMP (not a real problem) |
| Loss% increases and stays high | Real packet loss — investigate that router |
| Latency spike at specific hop | That router is slow or congested |

---

# Key Takeaways

- **`ping -c1`** is the fastest way to test connectivity, DNS, and routing together.
- **`ip a s`** shows all IP configuration (the modern `ifconfig`).
- **`ping6`** for IPv6 — remember the `%interface` for link-local addresses.
- **`ip r`** for IPv4 routing; **`ip -6 r`** for IPv6 routing.
- **`/etc/resolv.conf`** contains DNS server IPs and search domains.
- **`mtr`** is the ultimate troubleshooting tool — running statistics, not just one-off.
- Use `mtr --report -c 1000` to generate meaningful data for slow services.
- **`ss -plunt`** shows open ports AND which processes own them (always available, even on minimal installs).
- **`netstat`** works but may need installation; `ss` is the modern replacement.
- **`ip -s link show`** shows packet statistics (errors, drops) for troubleshooting physical issues.