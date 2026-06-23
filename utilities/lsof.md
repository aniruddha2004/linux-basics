# `lsof` — List Open Files

> Utility to see which processes have files, directories, devices, or network sockets open.

---

## What Is `lsof`?

`lsof` stands for **List Open Files**. In Linux, everything is a file — regular files, directories, network sockets, devices, pipes. `lsof` shows which process is using each one.

Useful for:
- Finding which process is using a file
- Checking which ports are listening
- Diagnosing "device or resource busy" errors
- Seeing network connections by process

---

## Common Commands

### List All Open Files

```bash
lsof
lsof | head -20
```

> This outputs every open file on the system. Usually too much, so filter it with `head`, `grep`, or specific flags.

### Files Opened by a Specific Process

```bash
lsof -p 72131
```

> Use `-p` followed by a PID. Great for inspecting what a running application has loaded.

### Processes Using a Specific File

```bash
lsof ~/.bashrc
lsof /var/log/auth.log
```

> If a file is open, this shows which process is holding it.

### Files Opened by a User

```bash
lsof -u aniruddha_ghosh
```

> Shows every file opened by the specified user.

### Network Connections

```bash
lsof -i :80          # Processes using port 80
lsof -i tcp          # All TCP connections
lsof -i udp          # All UDP connections
lsof -i              # All network connections
```

> `-i` stands for internet (network) files.

---

## Output Columns Explained

| Column | Meaning |
|--------|---------|
| `COMMAND` | Name of the command/process |
| `PID` | Process ID |
| `TID` | Thread ID (shown if threaded) |
| `TASKCMD` | Task command name |
| `USER` | User running the process |
| `FD` | File descriptor. Common values: `cwd` (current dir), `rtd` (root dir), `txt` (program code), `mem` (memory-mapped file), number followed by `r/w/u` (fd + mode) |
| `TYPE` | File type: `REG` (regular), `DIR` (directory), `CHR` (character device), `IPv4`, `IPv6` |
| `DEVICE` | Device number where the file lives |
| `SIZE/OFF` | File size or current offset |
| `NODE` | Inode number |
| `NAME` | File path, port, or socket name |

---

## File Descriptor Letters

| Letter | Meaning |
|--------|---------|
| `r` | Read access |
| `w` | Write access |
| `u` | Read/Write access |
| `cwd` | Current working directory |
| `rtd` | Root directory |
| `txt` | Program text/code |
| `mem` | Memory-mapped file |

---

## Quick Reference

| Task | Command |
|------|---------|
| List all open files | `lsof` |
| Files by process | `lsof -p <PID>` |
| Processes by file | `lsof <file>` |
| Files by user | `lsof -u <username>` |
| Port usage | `lsof -i :<port>` |
| TCP connections | `lsof -i tcp` |
| UDP connections | `lsof -i udp` |

