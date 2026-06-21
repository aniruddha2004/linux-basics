# Linux Notes

> Personal notes taken while learning Linux fundamentals. Organized, rephrased, and maintained for easy reference.

---

## Table of Contents

| Topic | File |
|-------|------|
| Basic Commands | [`basic-commands.md`](./basic-commands.md) |
| Standard Streams (STDIN/STDOUT/STDERR) | [`standard-streams.md`](./standard-streams.md) |
| APT Package Management | [`apt-package-management.md`](./apt-package-management.md) |
| File Permissions | [`file-permissions.md`](./file-permissions.md) |
| User Account Management | [`user-management.md`](./user-management.md) |

---

## File Overview

### [basic-commands.md](./basic-commands.md)

Essential Linux commands for everyday navigation and file management.

**Covers:**
- Navigation: `pwd`, `ls`, `cd`
- File Management: `touch`, `cat`, `less`, `more`, `head`, `tail`, `wc`, `cut`, `sort`, `grep`, `cp`, `mv`, `rm`
- Directory Operations: `mkdir`, `rmdir`
- System Information: `whoami`, `who`, `uname`, `hostname`, `clear`, `date`, `uptime`
- Help & Manuals: `man`, `info`, `whatis`, `which`, `type`, `alias`

### [standard-streams.md](./standard-streams.md)

Understanding how Linux handles input/output streams and redirection.

**Covers:**
- The three standard streams: STDIN (0), STDOUT (1), STDERR (2)
- Redirection operators: `>`, `>>`, `2>`, `&>`, `<`
- The pipe operator `|` and how it works with streams
- Logical AND `&&` and exit status vs stderr
- File descriptors cheat sheet

### [apt-package-management.md](./apt-package-management.md)

Package management on Debian-based systems using APT.

**Covers:**
- `apt-get` commands: `update`, `upgrade`, `install`, `remove`, `purge`, `autoremove`, `clean`
- `apt-cache` commands: `search`, `show`, `policy`, `depends`, `rdepends`
- Repository configuration: `/etc/apt/sources.list` and `sources.list.d/`
- Adding repositories and GPG keys (legacy and modern methods)
- Complete example: Adding Docker repository

### [file-permissions.md](./file-permissions.md)

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

### [user-management.md](./user-management.md)

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

## Quick Reference

| Need to... | Check this file |
|------------|----------------|
| Look up a command or its flags | [`basic-commands.md`](./basic-commands.md) |
| Understand redirection or pipes | [`standard-streams.md`](./standard-streams.md) |
| Install/remove/update packages | [`apt-package-management.md`](./apt-package-management.md) |
| Fix or set file permissions | [`file-permissions.md`](./file-permissions.md) |
| Manage users or understand /etc/passwd | [`user-management.md`](./user-management.md) |

---

> **Last Updated:** June 21, 2026
