# Linux Notes

> Personal notes taken while learning Linux fundamentals. Organized, rephrased, and maintained for easy reference.

---

## Table of Contents

| Topic | File |
|-------|------|
| Basic Commands | [`basics/basic-commands.md`](./basics/basic-commands.md) |
| Standard Streams (STDIN/STDOUT/STDERR) | [`basics/standard-streams.md`](./basics/standard-streams.md) |
| APT Package Management | [`system-admin/apt-package-management.md`](./system-admin/apt-package-management.md) |
| User Account Management | [`system-admin/user-management.md`](./system-admin/user-management.md) |
| Filesystem Overview | [`filesystem/filesystem-overview.md`](./filesystem/filesystem-overview.md) |
| File Permissions | [`filesystem/file-permissions.md`](./filesystem/file-permissions.md) |
| Process Management | [`process-management/process-theory.md`](./process-management/process-theory.md) |
| Process Commands | [`process-management/process-commands.md`](./process-management/process-commands.md) |
| `top` / `htop` | [`process-management/htop-top.md`](./process-management/htop-top.md) |
| `/proc` Filesystem | [`process-management/proc-filesystem.md`](./process-management/proc-filesystem.md) |

---

## Directory Structure

```
Linux/
├── basics/                     # Everyday commands and concepts
│   ├── basic-commands.md
│   └── standard-streams.md
├── system-admin/               # Administration and configuration
│   ├── apt-package-management.md
│   └── user-management.md
├── filesystem/                 # Filesystem hierarchy and permissions
│   ├── filesystem-overview.md
│   └── file-permissions.md
├── process-management/         # Processes, threads, and monitoring
│   ├── process-theory.md
│   ├── process-commands.md
│   ├── htop-top.md
│   └── proc-filesystem.md
└── README.md
```

---

## File Overview

### [basics/basic-commands.md](./basics/basic-commands.md)

Essential Linux commands for everyday navigation and file management.

**Covers:**
- Navigation: `pwd`, `ls`, `cd`
- File Management: `touch`, `cat`, `less`, `more`, `head`, `tail`, `wc`, `cut`, `sort`, `grep`, `cp`, `mv`, `rm`
- Directory Operations: `mkdir`, `rmdir`
- System Information: `whoami`, `who`, `uname`, `hostname`, `clear`, `date`, `uptime`
- Help & Manuals: `man`, `info`, `whatis`, `which`, `type`, `alias`

---

### [basics/standard-streams.md](./basics/standard-streams.md)

Understanding how Linux handles input/output streams and redirection.

**Covers:**
- The three standard streams: STDIN (0), STDOUT (1), STDERR (2)
- Redirection operators: `>`, `>>`, `2>`, `&>`, `<`
- The pipe operator `|` and how it works with streams
- Logical AND `&&` and exit status vs stderr
- File descriptors cheat sheet

---

### [system-admin/apt-package-management.md](./system-admin/apt-package-management.md)

Package management on Debian-based systems using APT.

**Covers:**
- `apt-get` commands: `update`, `upgrade`, `install`, `remove`, `purge`, `autoremove`, `clean`
- `apt-cache` commands: `search`, `show`, `policy`, `depends`, `rdepends`
- Repository configuration: `/etc/apt/sources.list` and `sources.list.d/`
- Adding repositories and GPG keys (legacy and modern methods)
- Complete example: Adding Docker repository

---

### [system-admin/user-management.md](./system-admin/user-management.md)

Basics of managing user accounts on Linux.

**Covers:**
- Core user files: `/etc/passwd`, `/etc/shadow`, `/etc/group`
- Field-by-field breakdown of each file
- Creating users: `useradd` vs `adduser`
- Setting passwords: `passwd`
- Modifying users: `usermod`
- Deleting users: `userdel`
- `/etc/skel` and default home files
- Password locking/unlocking

---

### [filesystem/filesystem-overview.md](./filesystem/filesystem-overview.md)

Overview of the Linux directory hierarchy.

**Covers:**
- All standard directories: `/bin`, `/boot`, `/dev`, `/etc`, `/home`, `/lib`, `/lib64`, `/media`, `/mnt`, `/opt`, `/proc`, `/root`, `/sbin`, `/tmp`, `/usr`, `/var`
- What each directory is meant for
- Dotfiles in home directories (`.bashrc`, `.profile`, `.bash_logout`)
- Where they come from (`/etc/skel/`)
- Key distinctions between similar directories

---

### [filesystem/file-permissions.md](./filesystem/file-permissions.md)

Understanding permission bits, ownership, and access control.

**Covers:**
- Permission string breakdown: `rwxrwxrwx`
- File types: `-`, `d`, `l`, `c`, `b`, `s`, `p`
- `rwx` differences between files and directories
- Numeric and symbolic `chmod`
- `chown` and `chgrp`
- Special bits: SUID, SGID, Sticky bit
- Default permissions and `umask`

---

### [process-management/process-theory.md](./process-management/process-theory.md)

Core concepts behind process management in Linux.

**Covers:**
- What is a process and its components
- Process lifecycle and states (Running, Ready, Blocked, Zombie)
- Threads vs processes — differences and trade-offs
- LWPs (Lightweight Processes) and how Linux implements threads
- PIDs, TIDs, and Thread Group IDs
- How `ps -efL` shows processes and threads
- Interpreting your `opencode` example with 28 threads
- Signals: SIGTERM vs SIGKILL, SIGSTOP/SIGCONT, SIGINT

---

### [process-management/process-commands.md](./process-management/process-commands.md)

Commands for inspecting and controlling processes.

**Covers:**
- `ps` flags: `a`, `u`, `x`, `e`, `f`, `L` and their combinations
- `kill` and `pkill` — differences and use cases
- `pgrep` — finding PIDs by name
- `nice` and `renice` — adjusting process priority
- Signal numbers and when to use each

---

### [process-management/htop-top.md](./process-management/htop-top.md)

Interactive process viewers and resource monitoring.

**Covers:**
- `top` key bindings and columns
- `htop` interactive controls and visual features
- Column meanings: PID, USER, PRI, NI, VIRT, RES, SHR, S, CPU%, MEM%, TIME+
- PRI vs NI — the difference between kernel priority and user-controllable nice value
- VIRT vs RES vs SHR — what each memory metric actually means
- MEM% calculation and why it matters
- `top` vs `htop` comparison

---

### [process-management/proc-filesystem.md](./process-management/proc-filesystem.md)

The `/proc` pseudo-filesystem — kernel data exposed as files.

**Covers:**
- What `/proc` is and how it works (virtual filesystem)
- Top-level files: `cpuinfo`, `meminfo`, `loadavg`, `version`, `uptime`
- Per-process directories (`/proc/<PID>/`)
- Key files: `cmdline`, `status`, `statm`, `environ`, `fd/`, `cwd`, `exe`
- Reading process memory and state information
- Real example: Inspecting PID 1 (systemd)

---

## Quick Reference

| Need to... | Check this file |
|------------|----------------|
| Look up a command or its flags | [`basics/basic-commands.md`](./basics/basic-commands.md) |
| Understand redirection or pipes | [`basics/standard-streams.md`](./basics/standard-streams.md) |
| Install/remove/update packages | [`system-admin/apt-package-management.md`](./system-admin/apt-package-management.md) |
| Manage users or understand /etc/passwd | [`system-admin/user-management.md`](./system-admin/user-management.md) |
| Understand Linux directory structure | [`filesystem/filesystem-overview.md`](./filesystem/filesystem-overview.md) |
| Fix or set file permissions | [`filesystem/file-permissions.md`](./filesystem/file-permissions.md) |
| Understand processes, threads, and PIDs | [`process-management/process-theory.md`](./process-management/process-theory.md) |
| Control processes (kill, renice, etc.) | [`process-management/process-commands.md`](./process-management/process-commands.md) |
| Monitor processes interactively | [`process-management/htop-top.md`](./process-management/htop-top.md) |
| Inspect `/proc` for process/kernel info | [`process-management/proc-filesystem.md`](./process-management/proc-filesystem.md) |

---

> **Last Updated:** June 21, 2026
