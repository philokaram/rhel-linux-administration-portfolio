# Tuning System Performance — Notes

## 1. What Is Tuned?

- **tuned** = a daemon that simplifies system performance optimization
- Runs quietly in the background
- Applies **predefined profiles** to match specific workload requirements
- A **Red Hat innovation** adopted by many other Linux distributions

### Why use tuned?

| Problem | Solution |
|---------|----------|
| Manually tweaking dozens of low-level settings | Predefined, validated profiles |
| Different workloads need different settings | Dynamic profile switching |
| Configuration drift | Declarative, centralized management |

---

## 2. Tuning Profiles — Performance Modes

### Common profiles:

| Profile | Use Case | Trade-off |
|---------|----------|-----------|
| `throughput-performance` | Servers, high performance | Increased power consumption |
| `virtual-guest` | Virtual machines (guest) | Optimized for virtualization |
| `virtual-host` | Hypervisor (host) | Optimized for running VMs |
| `balanced` | General non-specialized tuning | Middle ground |
| `powersave` | Laptops, power conservation | Lower performance |
| `latency-performance` | Low latency workloads | Power/throughput trade-off |
| `network-latency` | Network-sensitive apps | CPU/network trade-off |

> 💡 **Analogy:** Lamborghini (high performance, high fuel consumption) vs Toyota Prius (fuel-efficient, less performance). You can't have it both ways.

---

## 3. Installing and Verifying tuned

### Check if tuned is installed:

```bash
rpm -q tuned
```

### Check tuned service status:

```bash
systemctl status tuned
```

> 💡 tuned is installed and running by default on RHEL.

---

## 4. Managing Profiles — `tuned-adm`

### View active profile:

```bash
tuned-adm active
```

### Get recommended profile:

```bash
tuned-adm recommend
```

### List all available profiles:

```bash
tuned-adm list
```

or

```bash
tuned-adm profile
```

### Switch to a profile:

```bash
tuned-adm profile throughput-performance
```

### Verify the change:

```bash
tuned-adm active
```

---

## 5. Dynamic Tuning — Automatic Profile Switching

### Concept:

- tuned can **monitor system activity**
- If heavy CPU usage is detected → switch to high-performance profile
- When workload drops → switch back to balanced/powersave

### Enable dynamic tuning:

```bash
vi /etc/tuned/tuned-main.conf
```

**Line to uncomment/set:**
```
dynamic_tuning = 1
```

> 💡 This is not enabled by default. Check the man page for `tuned-main.conf` for details.

---

## 6. Profile Storage Locations

| Location | Purpose |
|----------|---------|
| `/usr/lib/tuned/profiles/` | **RPM-provided** profiles — **do not edit** |
| `/etc/tuned/profiles/` | **Custom** profiles — create your own here |

> 💡 **Best practice:** Never edit files in `/usr/lib/` — they are owned by RPMs and will be overwritten on updates.

---

## 7. Creating a Custom Profile

### Step 1 — Copy an existing profile:

```bash
rsync -a /usr/lib/tuned/profiles/powersave/ /etc/tuned/profiles/custom_powersave/
```

### Step 2 — Edit the profile:

```bash
vi /etc/tuned/profiles/custom_powersave/tuned.conf
```

**Example customizations:**
```
[main]
summary=Custom power-saving profile

[sysctl]
vm.dirty_ratio = 65
vm.dirty_writeback_centisecs = 4500
```

### Step 3 — Verify the profile appears:

```bash
tuned-adm list
```

**Output includes:** `custom_powersave`

> 💡 No need to restart tuned — it automatically detects new profiles.

---

## 8. Applying a Custom Profile

```bash
tuned-adm profile custom_powersave
```

### Verify tunables took effect:

```bash
sysctl vm.dirty_ratio
sysctl vm.dirty_writeback_centisecs
```

---

## 9. Viewing Sysctl Settings

### View a specific setting:

```bash
sysctl vm.dirty_ratio
```

### View all settings:

```bash
sysctl -a
```

### Common tunables:

| Tunable | Purpose |
|---------|---------|
| `vm.dirty_ratio` | Percentage of memory before writes are flushed to disk |
| `vm.dirty_writeback_centisecs` | How often kernel checks for dirty pages |
| `vm.swappiness` | Tendency to swap |
| `kernel.sched_migration_cost_ns` | CPU load balancing sensitivity |

> 📌 Full coverage of these tunables is in the RH442 course (Performance Tuning).

---

## 10. Managing tuned via Web Console (Cockpit)

### Steps:

1. Connect to `https://servera:9090`
2. Log in (e.g., `student`)
3. Turn on administrative access
4. On the **Overview** page, find **Configuration**
5. Click on the current profile
6. Select a new profile
7. Click **Change profile**

### Example:

```
Current: balanced → Click → Select: throughput-performance → Change
```

> 💡 This is equivalent to `tuned-adm profile throughput-performance`.

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Check tuned package | `rpm -q tuned` |
| Check tuned service | `systemctl status tuned` |
| View active profile | `tuned-adm active` |
| View recommended profile | `tuned-adm recommend` |
| List all profiles | `tuned-adm list` |
| Switch profile | `tuned-adm profile throughput-performance` |
| View sysctl setting | `sysctl vm.dirty_ratio` |
| View all sysctl settings | `sysctl -a` |
| View tuned config file | `cat /etc/tuned/tuned-main.conf` |
| View profile storage | `ls /usr/lib/tuned/profiles/` |
| Copy profile for customization | `rsync -a /usr/lib/tuned/profiles/powersave/ /etc/tuned/profiles/custom_powersave/` |
| Web console | `https://server:9090` → Overview → Configuration |

---

# Profile Storage — System vs Custom

```
/usr/lib/tuned/profiles/
├── balanced/
│   └── tuned.conf          # System-provided (do not edit)
├── powersave/
│   └── tuned.conf          # System-provided (do not edit)
├── throughput-performance/
│   └── tuned.conf          # System-provided (do not edit)
└── ...

/etc/tuned/profiles/
└── custom_powersave/       # Your custom profile (edit here)
    └── tuned.conf
```

---

# Key Takeaways

- **tuned** = daemon for automatic system performance tuning.
- Uses **predefined profiles** for different workloads.
- **`tuned-adm active`** shows current profile.
- **`tuned-adm list`** shows all available profiles.
- **`tuned-adm profile <name>`** switches profiles.
- **Dynamic tuning** allows automatic switching based on workload.
- **Custom profiles** go in `/etc/tuned/profiles/` (not `/usr/lib/`).
- **Profile settings** often include sysctl tunables.
- **Web Console** provides a GUI for changing profiles.
- **Trade-off:** Performance vs power consumption vs latency.
- **Next step:** For deeper tuning, see RH442 (Performance Tuning).