# RHEL Linux Administration Portfolio

## 📚 Repository Overview

This repository contains comprehensive notes from Red Hat System Administration I (RH124) and Red Hat System Administration II (RH134) courses. It serves as a practical portfolio demonstrating hands-on Linux system administration skills.

---

## 🗂️ Repository Structure

### 📁 RH124 - Red Hat System Administration I

#### 1. [00-intro](./RH124-system-admin-I/00-intro/)
Introduction to Red Hat Enterprise Linux
- `chapter-1-linux-intro/chapter-notes.md` - Course orientation and RHEL overview

---

#### 2. [01-command-line](./RH124-system-admin-I/01-command-line/)
**Chapter 2: Accessing the Command Line**

- **Section 2.1:** Accessing the Command Line in a Text Interface
  - SSH and TTY explanations with diagrams
  - `notes.md` - Command-line access methods

- **Section 2.2:** Accessing the Command Line with the Desktop Environment
  - GNOME desktop and application grid navigation
  - `notes.md` - Desktop environment usage

- **Section 2.3:** Executing Commands with the Bash Shell
  - `notes.md` - Basic Bash shell operations

- **Lab:** [lab.md](./RH124-system-admin-I/01-command-line/chapter-2-accessing-the-command-line/lab/lab.md) - Practical command-line exercises

---

#### 3. [02-help-and-documentation](./RH124-system-admin-I/02-help-and-documentation/)
**Chapters 3-5: Getting Help and Documentation**

- **Chapter 3:** Getting Help from Local Documentation
  - `exercise.md` - Manual pages practice
  - `lab.md` - Documentation exercises
  - `notes.md` - Using man pages and help commands

- **Chapter 4:** Registering Systems for Red Hat Support
  - `notes.md` - System registration with Red Hat

- **Chapter 5:** Getting AI-assisted Help with RHEL Lightspeed
  - `notes.md` - Using AI command-line assistant

---

#### 4. [03-filesystem](./RH124-system-admin-I/03-filesystem/)
**Chapter 6: Navigating the Filesystem Hierarchy**
- `hierarchy.svg` - Filesystem hierarchy diagram
- `notes-1.md` - Linux filesystem structure
- `notes-2.md` - File naming and path specifications

---

#### 5. [04-file-management](./RH124-system-admin-I/04-file-management/)
**Chapters 7-9: File Management**

- **Chapter 7:** Managing Files from the Command Line
  - **Section 7.1:** Managing Files with Command-line Tools
    - `exercies.md`, `notes.md` - cp, mv, rm, mkdir, etc.
  - **Section 7.2:** Creating Links Between Files
    - `exercise.md`, `notes.md` - Hard and soft links
  - **Section 7.3:** Matching File Names with Shell Expansions
    - `notes.md` - Wildcards and globbing
  - `lab.md` - Comprehensive file management lab

- **Chapter 8:** Editing Text Files
  - `notes.md` - Vim and text editors

- **Chapter 9:** Redirecting Shell Input and Output
  - `notes.md` - Pipes, redirections, and streams

---

#### 6. [05-users-and-groups](./RH124-system-admin-I/05-users-and-groups/)
**Chapter 10: Managing Local Users and Groups**
- `notes-1.md` - Users and groups concepts
- `notes-2.md` - Superuser access (sudo)
- `notes-3.md` - Managing user accounts
- `notes-4.md` - Managing group accounts
- `notes-5.md` - User password management

---

#### 7. [06-permissions](./RH124-system-admin-I/06-permissions/)
**Chapter 11: Controlling Access to Files**
- `notes-1.md` - File permissions (rwx)
- `notes-2.md` - Managing permissions from command line
- `notes-3.md` - Default permissions and special permissions

---

#### 8. [07-software-management](./RH124-system-admin-I/07-software-management/)
**Chapters 12-14: Software Management**

- **Chapter 12:** Installing and Updating Software with RPM
  - `notes-1.md` - RPM package investigation
  - `notes-2.md` - DNF package management
  - `notes-3.md` - DNF repositories

- **Chapter 13:** Installing and Updating Applications with Flatpak
  - `notes-1.md` - Flatpak configuration
  - `notes-2.md` - Managing Flatpak applications

- **Chapter 14:** Accessing Removable Media
  - `notes-1.md` - Filesystems and block devices
  - `notes-2.md` - Mounting/unmounting
  - `noes-3.md` - Locating files

---

#### 9. [08-processes-services](./RH124-system-admin-I/08-processes-services/)
**Chapters 15-16: Processes and Services**

- **Chapter 15:** Monitoring and Managing Linux Processes
  - `notes-1.md` - Process lifecycle
  - `notes-2.md` - Job control
  - `notes-3.md` - Sending signals
  - `notes-4.md` - Monitoring with top/ps

- **Chapter 16:** Controlling Services and Daemons
  - `notes-1.md` - Systemd services
  - `notes-2.md` - Service management

---

#### 10. [09-networking-ssh](./RH124-system-admin-I/09-networking-ssh/)
**Chapters 17-19: Networking and SSH**

- **Chapter 17:** Introduction to Networking
  - `notes-1.md` - Network concepts
  - `notes-2.md` - Validating configuration

- **Chapter 18:** Managing Network Configuration
  - `notes-1.md` - Command-line network config
  - `notes-2.md` - Network configuration files
  - `notes-3.md` - Hostnames and name resolution

- **Chapter 19:** Configuring and Securing SSH
  - `notes-1.md` - SSH host keys
  - `notes-2.md` - SSH key-based authentication

---

### 📁 RH134 - Red Hat System Administration II

#### 1. [01-bash-scripting](./RH134-system-admin-II/01-bash-scripting/)
**Chapter 1: Shell Scripting and the Command Line**
- `notes-1.md` - Shell environment customization
- `notes-2.md` - Writing simple Bash scripts
- `notes-3.md` - Loops and conditional commands

---

#### 2. [02-regex](./RH134-system-admin-II/02-regex/)
**Chapter 2: Using Regular Expressions for Practical Applications**
- `notes.md` - Regex fundamentals and grep

---

#### 3. [03-task-scheduling](./RH134-system-admin-II/03-task-scheduling/)
**Chapters 3-4: Task Scheduling**

- **Chapter 3:** Scheduling User Tasks
  - `notes-1.md` - One-time jobs with at
  - `notes-2.md` - Recurring jobs with crontab

- **Chapter 4:** Scheduling System Tasks
  - `notes-1.md` - Systemd timer units
  - `notes-2.md` - Temporary file management
  - `notes-3.md` - System cron jobs

---

#### 4. [04-logging](./RH134-system-admin-II/04-logging/)
**Chapter 5: Analyzing and Storing Logs**
- `notes-1.md` - System log architecture
- `notes-2.md` - Syslog events
- `notes-3.md` - System journal (journalctl)
- `notes-4.md` - Persistent journal
- `notes-5.md` - Time synchronization (chrony)

---

#### 5. [05-selinux](./RH134-system-admin-II/05-selinux/)
**Chapter 6: Managing Security with SELinux**
- `notes-1.md` - SELinux operating modes
- `notes-2.md` - File contexts management
- `notes-3.md` - SELinux booleans
- `notes-4.md` - Troubleshooting SELinux issues

---

#### 6. [06-storage-lvm](./RH134-system-admin-II/06-storage-lvm/)
**Chapters 7-10: Storage Management**

- **Chapter 7:** Archiving Files
  - `notes.md` - Compressed tar archives

- **Chapter 8:** Transferring Files
  - `notes-1.md` - File transfer (scp, rsync)
  - `notes-2.md` - Content synchronization

- **Chapter 9:** Tuning System Performance
  - `notes-1.md` - Tuning profiles (tuned)
  - `notes-2.md` - Process scheduling (nice, renice)

- **Chapter 10:** Managing Basic Storage
  - `notes-1.md` - Partitions and filesystems
  - `notes-2.md` - Swap space management

---

#### 7. [07-boot-troubleshooting](./RH134-system-admin-II/07-boot-troubleshooting/)
**Chapters 11-13: Boot and Troubleshooting**

- **Chapter 11:** Managing Storage with LVM
  - `notes-1.md` - Creating logical volumes
  - `notes-2.md` - Extending logical volumes
  - `notes-3.md` - LVM management

- **Chapter 12:** Controlling and Troubleshooting the Boot Process
  - `notes-1.md` - Boot loader and kernel
  - `notes-2.md` - Boot targets
  - `notes-3.md` - Repairing filesystems

- **Chapter 13:** Recovering Superuser Access
  - `notes.md` - Resetting root password

---

#### 8. [08-firewall-security](./RH134-system-admin-II/08-firewall-security/)
**Chapters 14-16: Network Security**

- **Chapter 14:** Managing Network Security
  - `notes-1.md` - Firewall (firewalld)
  - `notes-2.md` - SELinux port labeling

- **Chapter 15:** Accessing Network-attached Storage
  - `notes-1.md` - NFS mounting
  - `notes-2.md` - Autofs automounting

- **Chapter 16:** Installing Red Hat Enterprise Linux
  - `notes-1.md` - Interactive installation
  - `notes-2.md` - Kickstart automation

---

#### 9. [09-podman-containers](./RH134-system-admin-II/09-podman-containers/)
**Chapters 17-18: Containers**

- **Chapter 17:** Managing Containers with Podman
  - `notes-1.md` - Container introduction
  - `notes-2.md` - Running containers
  - `notes-3.md` - Creating container images

- **Chapter 18:** Working with Image-based RHEL
  - `notes-1.md` - Image mode overview
  - `notes-2.md` - Creating installable images
  - `notes-3.md` - Installing RHEL in image mode
  - `notes-4.md` - Managing image-based systems

---

## 📖 Course Content Summary

### RH124 - Red Hat System Administration I
- ✅ Introduction to RHEL
- ✅ Command-line access and Bash shell
- ✅ Getting help (man pages, documentation)
- ✅ System registration
- ✅ Filesystem navigation
- ✅ File management and editing
- ✅ I/O redirection
- ✅ User and group management
- ✅ File permissions and access control
- ✅ Software management (RPM, DNF, Flatpak)
- ✅ Removable media
- ✅ Process and service management
- ✅ Networking fundamentals and configuration
- ✅ SSH configuration and security

### RH134 - Red Hat System Administration II
- ✅ Shell scripting
- ✅ Regular expressions
- ✅ Task scheduling (at, cron, systemd timers)
- ✅ Logging and analysis
- ✅ SELinux security
- ✅ Archiving and file transfer
- ✅ System performance tuning
- ✅ Storage management (partitions, LVM)
- ✅ Boot process and troubleshooting
- ✅ Firewall management
- ✅ Network-attached storage
- ✅ RHEL installation (interactive and Kickstart)
- ✅ Container management with Podman
- ✅ Image-based RHEL deployment

---

## 🛠️ Technologies and Tools Covered

| Category | Technologies |
|----------|--------------|
| **OS** | Red Hat Enterprise Linux 9/10 |
| **Shell** | Bash |
| **Package Management** | RPM, DNF, Flatpak |
| **Process/Services** | systemd |
| **Networking** | nmcli, firewalld, SSH |
| **Storage** | LVM, NFS, autofs |
| **Security** | SELinux, sudo |
| **Monitoring** | top, ps, journalctl |
| **Automation** | Bash scripting, Kickstart |
| **Containers** | Podman |

---

## 🎯 Key Skills Demonstrated

1. **System Administration**: User/group management, permissions, services
2. **Storage Management**: Partitions, LVM, filesystems, swap
3. **Security**: SELinux configuration, firewall rules, SSH hardening
4. **Automation**: Bash scripts, task scheduling, Kickstart
5. **Networking**: Configuration, troubleshooting, NFS, SSH
6. **Containers**: Podman management, image creation
7. **Troubleshooting**: Boot process, logs, SELinux, performance
8. **Software Management**: RPM, DNF, Flatpak repositories

---

## 🚀 Getting Started

```bash
# Clone the repository
git clone https://github.com/philokaram/rhel-linux-administration-portfolio.git

# Navigate to specific chapters
cd rhel-linux-administration-portfolio/RH124-system-admin-I/01-command-line

# Review notes and practice labs
cat notes.md
cat lab/lab.md
```

---


**Maintained by:** Philopateer Karam  
**Last Updated:** 23 / 6 / 2026
