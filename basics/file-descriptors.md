# File Descriptors & "Everything is a File"

> Foundational Linux concepts: how processes access resources through file descriptors, and why so many things look like files.

---

## What Is a File Descriptor?

A **file descriptor (FD)** is a small integer that a process uses to refer to an open resource.

Think of it as a ticket number:

```text
Program asks to open resource
        │
        ▼
Kernel stores resource information
        │
        ▼
Kernel returns a file descriptor (e.g., 3)
```

Example in C:

```c
int fd = open("notes.txt", O_RDONLY);
```

The kernel might return `fd = 3`. From then on, the program uses `3` to read from or close the file.

```c
read(fd, buffer, 100);   // Read using FD
close(fd);               // Close FD when done
```

---

## Why Do File Descriptors Exist?

Instead of having separate APIs for files, sockets, pipes, terminals, and devices, Linux exposes them through a common interface:

```c
read(fd, ...)
write(fd, ...)
close(fd)
```

This uniformity makes it easier to write programs and connect different resources together.

---

## "Everything is a File"

In Linux, many different resources can be accessed through the file abstraction:

| Resource | File-like? |
|----------|------------|
| Regular file | Yes |
| Directory | Yes |
| Terminal | Yes |
| Pipe | Yes |
| Network socket | Yes |
| Device (`/dev/null`, `/dev/urandom`) | Yes |

This is why the same system calls (`read`, `write`, `close`) can work on many different types of resources.

---

## Standard File Descriptors

Every process starts with three predefined file descriptors:

| FD | Name | Purpose | Default Connection |
|----|------|---------|--------------------|
| `0` | **stdin** | Standard input | Keyboard |
| `1` | **stdout** | Standard output | Terminal screen |
| `2` | **stderr** | Standard error | Terminal screen |

---

## How Redirection Works

```bash
echo hello > out.txt
```

`echo` always writes to its stdout (FD 1). The shell intercepts this and changes where FD 1 points:

```text
Before:
  FD 1 -> Terminal

After redirection:
  FD 1 -> out.txt
```

The `echo` program does not know the difference — it simply writes to FD 1.

> More examples in [`standard-streams.md`](./standard-streams.md).

---

## How Pipes Work

```bash
ls | grep txt
```

The shell creates a pipe and connects the two processes:

```text
ls stdout (FD 1)
      │
      ▼
   [pipe]
      │
      ▼
grep stdin (FD 0)
```

Output from `ls` becomes input for `grep`. Neither program knows about the pipe directly — they just use their standard FDs.

---

## File Descriptor Table

Each process maintains a table of open file descriptors:

```text
FD TABLE for process 1234

0 -> stdin (keyboard)
1 -> stdout (terminal)
2 -> stderr (terminal)
3 -> notes.txt
4 -> network socket
5 -> pipe
```

When the process calls `read(3, ...)`, the kernel checks the table and accesses the corresponding resource.

---

## Are Directories Files?

Yes. A directory is a special type of file.

A regular file contains data:

```text
Hello World
```

A directory contains mappings between filenames and file locations:

```text
notes.txt  -> inode 12345
photo.jpg  -> inode 67890
script.sh  -> inode 54321
```

The kernel treats directories as filesystem objects just like regular files.

---

## Current Working Directory (cwd)

When you run:

```bash
cd /home/user/projects
```

the shell stores that directory as the process's current working directory.

```text
Process
   │
   +--> Current Working Directory: /home/user/projects
```

Whenever you use a relative path:

```bash
cat file.txt
```

Linux resolves `file.txt` relative to the current working directory.

---

## Special FD Entries Seen in `lsof`

In `lsof` output you may see non-numeric FD values:

| Entry | Meaning |
|-------|---------|
| `cwd` | Current Working Directory |
| `rtd` | Root Directory |
| `txt` | Program executable being run |
| `mem` | Memory-mapped file (shared library, etc.) |

These are associated with the process but are not normal numbered file descriptors like `0`, `1`, `2`, `3`.

> More in [`utilities/lsof.md`](../utilities/lsof.md).

---

## Key Takeaway

A Linux process does not directly access files, sockets, pipes, terminals, or devices. Instead, it uses **file descriptors**:

```c
read(fd)
write(fd)
close(fd)
```

The kernel uses the file descriptor to decide which resource to access. This abstraction is what makes files, pipes, sockets, terminals, and devices work consistently across Linux.

---

## Quick Reference

| FD | Name | Default Target |
|----|------|----------------|
| `0` | stdin | Keyboard |
| `1` | stdout | Terminal |
| `2` | stderr | Terminal |

| Concept | Meaning |
|---------|---------|
| File Descriptor | Integer ticket for an open resource |
| Everything is a File | Many resources use the same read/write/close interface |
| Redirection | Shell changes where an FD points |
| Pipe | Connects stdout of one process to stdin of another |
| Directory | A special file containing filename-to-location mappings |
| `cwd` | Current working directory of a process |
