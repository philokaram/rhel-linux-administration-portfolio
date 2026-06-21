# Transferring Files Between Systems — Notes

## 1. Overview — SFTP vs SCP

| Feature | SFTP | SCP |
|---------|------|-----|
| Purpose | Interactive remote file management | Quick one-time transfers |
| Interactive | ✅ Yes (browse, delete, create) | ❌ No (single command) |
| Resumable transfers | ✅ Yes (`re-get`) | ❌ No |
| Multiple files | ✅ Yes (`mget`, `mput`) | ❌ No (one at a time) |
| Underlying protocol | SSH (secure) | SSH (secure) |
| Best for | Managing many files/directories | Quick single-file transfers |

> 💡 Both SFTP and SCP are powered by **SSH** — data is encrypted.

---

## 2. SFTP — Interactive File Management

### Connect to a remote system:

```bash
sftp user@serverb
```

### Without username (uses current user):

```bash
sftp serverb
```

### SFTP commands:

| Command | Meaning | Local Equivalent |
|---------|---------|------------------|
| `pwd` | Print remote working directory | `lpwd` |
| `ls` | List remote files | `lls` |
| `cd` | Change remote directory | `lcd` |
| `mkdir` | Create remote directory | `lmkdir` |
| `rm` | Remove remote file | (N/A) |
| `put` | **Upload** file (local → remote) | (N/A) |
| `get` | **Download** file (remote → local) | (N/A) |
| `mput` | Upload multiple files | (N/A) |
| `mget` | Download multiple files | (N/A) |
| `re-get` | Resume interrupted download | (N/A) |
| `exit` or `quit` | Exit SFTP | (N/A) |

> 💡 Local commands are prefixed with `l` (e.g., `lpwd`, `lcd`, `lls`).

---

## 3. SFTP Example — Upload a File

```bash
# Connect to serverb
sftp serverb

# Check current directories
lpwd        # Local: /home/student
pwd         # Remote: /home/student

# Create remote directory
mkdir data

# Change to remote directory
cd data

# Upload a file
put /etc/fstab

# Verify upload
ls
```

**Result:** `fstab` is now on the remote system.

---

## 4. SFTP Example — Download a File

```bash
# Create local directory
lmkdir data

# Change to local directory
lcd data

# Download a file from remote
get bigfile

# Verify download
lls
```

**Result:** `bigfile` is now on the local system.

---

## 5. SFTP — Resuming Interrupted Transfers

### The problem:

- Network connection drops during a large file transfer
- You've already transferred 7 GB of a 10 GB file

### Solution — `re-get`:

```bash
re-get bigfile
```

> 💡 SFTP picks up exactly where it left off — no need to start over.

---

## 6. SCP — Quick One-Time Transfers

### Syntax:

```bash
scp source destination
```

### Upload local file to remote:

```bash
scp bigfile serverb:/home/student/
```

### Download remote file to local:

```bash
scp serverb:/home/student/bigfile .
```

### Copy to a different location:

```bash
scp serverb:/home/student/bigfile /var/tmp/
```

### Specify a different user:

```bash
scp bigfile devops@serverb:/home/devops/
```

> ⚠️ **SCP limitation:** You cannot have both source and destination be remote (server-to-server copy from a third system). You must run SCP from one of the servers.

---

## 7. SCP — Disadvantages

### Disadvantage 1 — No resume:

- If connection drops, transfer fails entirely
- Must start over from the beginning

### Disadvantage 2 — Copies 100% every time:

- SCP copies the entire file, even if only a small part changed
- Inefficient for large files that change frequently

### Example:

```bash
# First transfer: 100 MB
scp bigfile serverb:

# Add 50 MB to bigfile (now 150 MB)
cat /usr/share/dict/words >> bigfile

# Second transfer: copies ALL 150 MB
scp bigfile serverb:
```

> 💡 **Comparison:** SCP is like cp over the network — simple but not optimized for changes.

---

## 8. SCP vs SFTP — Quick Comparison

| Feature | SCP | SFTP |
|---------|-----|------|
| Interactive | No | Yes |
| Single file transfer | Yes | Yes |
| Multiple file transfer | No (with wildcards) | Yes (`mget`, `mput`) |
| Resumable | No | Yes (`re-get`) |
| Directory transfer | Yes (`-r`) | Yes |
| Uses SSH | Yes | Yes |
| Best for | Quick one-off transfers | Interactive management, large files |

---

# Complete Command Reference Table

| Task | Command |
|------|---------|
| Connect to SFTP (current user) | `sftp serverb` |
| Connect to SFTP (specific user) | `sftp user@serverb` |
| Upload file (SFTP) | `put localfile` |
| Download file (SFTP) | `get remotefile` |
| Resume download (SFTP) | `re-get remotefile` |
| Upload multiple (SFTP) | `mput *.txt` |
| Download multiple (SFTP) | `mget *.txt` |
| Local command (SFTP) | `lcd`, `lpwd`, `lls`, `lmkdir` |
| Exit SFTP | `exit` or `quit` |
| SCP upload | `scp localfile serverb:/path/` |
| SCP download | `scp serverb:/path/file .` |
| SCP recursive directory | `scp -r /local/dir serverb:/path/` |
| SCP with different port | `scp -P 2222 file serverb:` |

---

# SCP vs SFTP — When to Use Which

```
┌─────────────────────────────────────────────────────────────────┐
│  Use SCP when:                                                  │
│  • You need a quick, one-time file transfer                    │
│  • You're familiar with the cp command                         │
│  • The file is small or network is reliable                    │
│  • You're scripting simple transfers                           │
├─────────────────────────────────────────────────────────────────┤
│  Use SFTP when:                                                 │
│  • You need to browse/manage remote directories                │
│  • You're transferring multiple files                          │
│  • You're dealing with large files (resume support)            │
│  • You want to interactively explore the remote filesystem    │
└─────────────────────────────────────────────────────────────────┘
```

---

# Key Takeaways

- **SFTP** = interactive remote file management over SSH.
  - Browse, create, delete, and transfer files.
  - Local commands: `lpwd`, `lcd`, `lls`, `lmkdir`.
  - **Resumable transfers** with `re-get`.
  - `mput` and `mget` for multiple files.

- **SCP** = quick, non-interactive file transfers over SSH.
  - Works like `cp` but over the network.
  - **No resume** — if connection drops, start over.
  - **Copies 100%** every time — inefficient for large/changing files.

- **Both use SSH** → encrypted, secure, can use key-based authentication.

- **SCP limitation:** Cannot copy directly between two remote systems from a third system.

- **SCP use case:** Quick one-off transfers of small files.

- **SFTP use case:** Managing many files, large transfers, resumable downloads.

- **Coming next:** `rsync` — a better alternative for efficient transfers.