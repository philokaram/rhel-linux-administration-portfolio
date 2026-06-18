# Configuring Time Synchronization with NTP — Notes

## 1. Why Time Synchronization Matters

Without accurate time:

| Problem | Consequence |
|---------|-------------|
| Authentication | Can fail (Kerberos, tickets) |
| Logs | Go out of order (troubleshooting becomes difficult) |
| Clusters | Can misbehave (OpenShift, Kubernetes) |

> 💡 Every Linux server needs accurate time.

---

## 2. NTP vs Chrony

| Term | Meaning |
|------|---------|
| **NTP** | Network Time Protocol (like HTTP, FTP) |
| **Chrony** | Modern NTP implementation on RHEL (replaces legacy `ntpd`) |
| **chronyd** | The chrony daemon |

---

## 3. Viewing Time Status — `timedatectl`

### Basic status:

```bash
timedatectl
```

**Output example:**
```
               Local time: Mon 2025-09-01 14:30:00 UTC
           Universal time: Mon 2025-09-01 14:30:00 UTC
                 RTC time: Mon 2025-09-01 14:30:00
                Time zone: UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

| Field | Meaning |
|-------|---------|
| Local time | System's current local time |
| Universal time | UTC time |
| RTC time | Real-Time Clock (hardware clock) |
| Time zone | Configured timezone |
| NTP service | Whether NTP is active |

> 💡 On RHEL installation, NTP is enabled by default (using a pool of NTP servers).

---

## 4. Setting the Timezone — `timedatectl set-timezone`

### List available timezones:

```bash
timedatectl list-timezones | grep New_York
```

### Set timezone:

```bash
timedatectl set-timezone America/New_York
```

### Verify:

```bash
timedatectl
```

> 💡 Timezone names use underscores (`_`) for spaces: `America/New_York`, `Europe/London`.

---

## 5. Setting Time Manually (If NTP Disabled)

### Step 1 — Disable NTP:

```bash
timedatectl set-ntp false
```

### Step 2 — Set the time:

```bash
timedatectl set-time "2025-09-01 14:30:00"
```

### Step 3 — Re-enable NTP:

```bash
timedatectl set-ntp true
```

> 💡 Always disable NTP before manually setting time, then re-enable it.

---

## 6. Chrony Configuration — `/etc/chrony.conf`

### View configuration (without comments):

```bash
decomment /etc/chrony.conf
```

**Example output:**
```
server classroom.example.com iburst
```

### Server vs Pool:

| Directive | Meaning | Use Case |
|-----------|---------|----------|
| `server` | Single NTP server | Specific known server |
| `pool` | **Pool** of NTP servers | Redundancy (if one fails, use another) |

### Recommended — use `pool` for reliability:

```
pool 0.rhel.pool.ntp.org iburst
pool 1.rhel.pool.ntp.org iburst
pool 2.rhel.pool.ntp.org iburst
pool 3.rhel.pool.ntp.org iburst
```

> 💡 `iburst` speeds up initial time synchronization.

---

## 7. Restarting Chrony After Changes

```bash
systemctl restart chronyd
```

### Verify status:

```bash
systemctl status chronyd
```

---

## 8. Checking Time Sources — `chronyc sources`

### Basic:

```bash
chronyc sources
```

### Verbose:

```bash
chronyc sources -v
```

**Output:**
```
MS Name/IP address          Stratum Poll Reach LastRx Last sample
===========================================================================
^* classroom.example.com           2   6   377    45   -0.123ms[-0.145ms] +/- 1.234ms
```

### Symbol meanings:

| Symbol | Meaning |
|--------|---------|
| `*` | **Current** (best) time source |
| `+` | Backup source (usable) |
| `-` | Valid but not used (better sources available) |
| `X` | Unreliable — chrony ignoring it |
| `~` | Unreliable — chrony ignoring it |
| `?` | Not usable yet (often new server) |

---

## 9. Understanding Chrony Output Columns

| Column | Meaning |
|--------|---------|
| **MS** | Symbol (`*` = best source) |
| **Name/IP** | Server address |
| **Stratum** | Distance from reference clock (1-15, 16 = unsynchronized) |
| **Poll** | Polling interval (power of 2 seconds; 6 = 64 seconds) |
| **Reach** | Reliability score (octal; `377` = all 8 polls successful) |
| **LastRx** | Seconds since last response |
| **Last sample** | Time offset; margin of error in brackets |

---

## 10. NTP Stratum Hierarchy

| Stratum | Description |
|---------|-------------|
| **Stratum 0** | Reference clock (Atomic clock, GPS, radio) |
| **Stratum 1** | Directly connected to Stratum 0 (primary servers) |
| **Stratum 2** | Synchronizes from Stratum 1 |
| **Stratum 3+** | Further away from reference clock |
| **Stratum 15** | Highest usable stratum |
| **Stratum 16** | **Unsynchronized** (lost reliable time source) |

> 💡 Lower stratum = closer to the reference clock = more accurate.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| View time status | `timedatectl` |
| List timezones | `timedatectl list-timezones` |
| Set timezone | `timedatectl set-timezone America/New_York` |
| Disable NTP (for manual time) | `timedatectl set-ntp false` |
| Set time manually | `timedatectl set-time "2025-09-01 14:30:00"` |
| Enable NTP | `timedatectl set-ntp true` |
| View chrony config | `cat /etc/chrony.conf` |
| View chrony config (no comments) | `decomment /etc/chrony.conf` |
| Restart chrony | `systemctl restart chronyd` |
| Check time sources | `chronyc sources` |
| Check time sources (verbose) | `chronyc sources -v` |

---

# Chrony Configuration — Common Directives

| Directive | Purpose | Example |
|-----------|---------|---------|
| `server` | Single NTP server | `server 172.25.254.254 iburst` |
| `pool` | Pool of NTP servers | `pool 0.rhel.pool.ntp.org iburst` |
| `iburst` | Speed up initial sync | (added to server/pool) |
| `allow` | Allow clients to sync from this server | `allow 172.25.250.0/24` |
| `local stratum 10` | Serve time even when disconnected | (for isolated networks) |

---

# Chrony Source Symbols — Quick Reference

| Symbol | Meaning | Action |
|--------|---------|--------|
| `*` | Best source | Using it for sync |
| `+` | Backup source | Used for initial sync |
| `-` | Not used | Better sources available |
| `X` / `~` | Unreliable | Chrony ignoring it |
| `?` | Not usable | Wait for initial sync |

---

# Key Takeaways

- **Chrony** is the modern NTP implementation on RHEL (replaces `ntpd`).
- **`timedatectl`** is the primary command for viewing/setting time and timezone.
- **Timezones** are set with `timedatectl set-timezone`.
- **Always disable NTP** before manually setting time: `timedatectl set-ntp false`.
- **Configuration** is in `/etc/chrony.conf` — use **`pool`** for redundancy, not just `server`.
- **Restart chrony** after config changes: `systemctl restart chronyd`.
- **`chronyc sources`** shows time sources and their status.
- **`*`** = the selected best time source.
- **Stratum** = distance from the reference clock (lower = better).
- **Reach `377`** = all polls successful (reliable).
- **`iburst`** speeds up initial synchronization.
- **Chrony** ensures accurate time for authentication, logging, and clustering.