# File Permissions in Linux

> Understanding permission bits, ownership, and access control for files and directories.

---

## The Permission String

When you run `ls -l`, each file shows a 10-character permission string:

```bash
-rwxr-xr--  1 user group  1234 Jan 15 10:30 script.sh
drwxr-xr-x  2 user group  4096 Jan 15 09:00 documents
lrwxrwxrwx  1 user group    12 Jan 15 08:00 link -> /path/to/target
```

```
┌─┬─────────┬─────────┬─────────┐
│ │  Owner  │  Group  │ Others  │
│ │  rwx    │  rwx    │  rwx    │
├─┼─────────┼─────────┼─────────┤
│d│ rwx     │ r-x     │ r--     │
└─┴─────────┴─────────┴─────────┘
 │   ↑         ↑         ↑
 │   │         │         └── Others: read only
 │   │         └──────────── Group: read + execute
 │   └────────────────────── Owner: read + write + execute
 └────────────────────────── File type: directory
```

---

## The Leading Character: File Types

The first character indicates the file type:

| Symbol | Type | Description |
|--------|------|-------------|
| `-` | Regular file | Normal files (text, binary, scripts, etc.) |
| `d` | Directory | Folder containing files and subdirectories |
| `l` | Symbolic link | Pointer to another file/directory |
| `c` | Character device | Special file for devices (terminal, keyboard, etc.) |
| `b` | Block device | Special file for block-based devices (hard drives, SSDs) |
| `s` | Socket | Network/IPC socket file |
| `p` | Named pipe (FIFO) | Inter-process communication pipe |

```bash
# Examples
-rw-r--r--   # Regular file
drwxr-xr-x   # Directory
lrwxrwxrwx   # Symbolic link
crw-rw----   # Character device (e.g., /dev/tty0)
brw-rw----   # Block device (e.g., /dev/sda)
srw-------   # Unix domain socket
prw-------   # Named pipe
```

---

## Permission Bits: r, w, x

### For Regular Files

| Permission | Symbol | Numeric | Meaning |
|------------|--------|---------|---------|
| Read | `r` | `4` | View file contents (`cat`, `less`) |
| Write | `w` | `2` | Modify or delete file contents |
| Execute | `x` | `1` | Run as a program/script |

```bash
-rw-r--r--  file.txt     # Owner can read/write; others can only read
-rwxr-xr-x  script.sh     # Owner full access; others read+execute
-rwx------  secret.bin    # Owner only; no access for anyone else
```

### For Directories

| Permission | Symbol | Meaning |
|------------|--------|---------|
| Read | `r` | List directory contents (`ls`) |
| Write | `w` | Create, delete, rename files inside |
| Execute | `x` | Enter/access the directory (`cd`) |

```bash
drwxr-xr-x  docs/         # Everyone can enter and list; only owner can modify
drwx--x--x  private/      # Others can enter but cannot list contents
drwxrwxrwt  /tmp/         # Sticky bit: everyone can write, but only delete own files
```

> **Key Difference**: For directories, `x` means you can **access** it (use files inside), while `r` only lets you **see the list** of filenames.

```bash
# Example: r-- without x
chmod 400 secret_dir/
ls secret_dir/            # Works: can see filenames
cat secret_dir/file.txt   # Fails: cannot access files inside
```

---

## Numeric (Octal) Representation

Each permission is a sum of its numeric values:

| Permission | Value |
|------------|-------|
| Read | 4 |
| Write | 2 |
| Execute | 1 |
| None | 0 |

**Calculate each triplet:**

| Permission | Calculation | Octal |
|------------|-------------|-------|
| `rwx` | 4+2+1 | **7** |
| `rw-` | 4+2+0 | **6** |
| `r-x` | 4+0+1 | **5** |
| `r--` | 4+0+0 | **4** |
| `-wx` | 0+2+1 | **3** |
| `-w-` | 0+2+0 | **2** |
| `--x` | 0+0+1 | **1** |
| `---` | 0+0+0 | **0** |

**Full examples:**

| Numeric | Permissions | Description |
|---------|-------------|-------------|
| `777` | `rwxrwxrwx` | Everyone has full access |
| `755` | `rwxr-xr-x` | Owner full; others read+execute (common for scripts/programs) |
| `644` | `rw-r--r--` | Owner read+write; others read-only (common for files) |
| `700` | `rwx------` | Owner only; completely private |
| `750` | `rwxr-x---` | Owner full; group read+execute; others nothing |
| `666` | `rw-rw-rw-` | Everyone can read and write |
| `640` | `rw-r-----` | Owner read+write; group read; others nothing |
| `600` | `rw-------` | Owner read+write only (private files like SSH keys) |
| `400` | `r--------` | Owner read only (immutable config files) |
| `711` | `rwx--x--x` | Owner full; others can enter but not list (dropbox style) |

---

## `chmod` — Change Permissions

### Method 1: Numeric (Octal)

Directly set all permissions at once.

```bash
chmod 755 script.sh           # rwxr-xr-x
chmod 644 file.txt            # rw-r--r--
chmod 700 private_key.pem     # rwx------
chmod 777 shared_folder/      # rwxrwxrwx
```

---

### Method 2: Symbolic (Text-Based)

Modify specific permissions without affecting others.

**Who:**

| Symbol | Means |
|--------|-------|
| `u` | User (owner) |
| `g` | Group |
| `o` | Others |
| `a` | All (u + g + o) |

**Operators:**

| Symbol | Action |
|--------|--------|
| `+` | Add permission |
| `-` | Remove permission |
| `=` | Set exact permission |

**Permissions:** `r`, `w`, `x`

---

#### Add Permissions (`+`)

```bash
chmod u+x script.sh           # Add execute for owner
chmod g+w file.txt            # Add write for group
chmod o+r document.pdf        # Add read for others
chmod a+x program             # Add execute for everyone
chmod +x script.sh            # Shortcut for a+x
```

---

#### Remove Permissions (`-`)

```bash
chmod u-w file.txt            # Remove write for owner
chmod g-x script.sh           # Remove execute for group
chmod o-r secret.doc          # Remove read for others
chmod a-x program             # Remove execute for everyone
```

---

#### Set Exact Permissions (`=`)

```bash
chmod u=rwx file.txt          # Owner gets rwx exactly
chmod g=rx directory/         # Group gets rx exactly (removes w if present)
chmod o=rx file.txt           # Others get rx exactly
chmod u=rw,g=r,o= file.txt    # Owner: rw, Group: r, Others: nothing
```

---

#### Combined Examples

```bash
chmod u+rwx,g+rx,o-rwx file.txt     # Owner full, group +rx, others remove all
chmod u=rwx,g=rx,o=rx script.sh     # Same as chmod 755
chmod u=rw,g=r,o=r file.txt         # Same as chmod 644
chmod u+x,g+x,o-w filename          # Add execute for u+g, remove write for others
```

---

### Recursive Changes

Apply permissions to a directory and everything inside it.

```bash
chmod -R 755 myproject/             # All files and dirs get 755
chmod -R u+rwX directory/           # Add read+write, and X (conditional execute)
```

> **Note**: `X` (capital X) sets execute only if it's already set somewhere or is a directory — safer than `x` for mixed file trees.

---

## `chown` — Change Ownership

Change the user and/or group that owns a file.

```bash
sudo chown user file.txt              # Change owner
sudo chown :group file.txt            # Change group only
sudo chown user:group file.txt        # Change both owner and group
sudo chown -R user:group directory/   # Recursive
```

```bash
sudo chown alice document.txt         # alice owns it
sudo chown alice:developers app.conf  # alice owns, developers group
sudo chown :staff notes.txt           # Just change group to staff
sudo chown $USER:$USER file.txt      # Current user owns it
```

---

## `chgrp` — Change Group

Shortcut to change only the group.

```bash
sudo chgrp developers file.txt
sudo chgrp -R staff shared_folder/
```

---

## Special Permission Bits

Beyond rwx, there are three special bits:

### SUID (`s` on owner execute) — Set User ID

When set on an executable, it runs with the **file owner's** privileges.

```bash
-rwsr-xr-x  /usr/bin/passwd      # Runs as root even when executed by normal user
```

```bash
chmod u+s program                # Set SUID
chmod 4755 program               # Same: 4 + 755
chmod u-s program                # Remove SUID
```

### SGID (`s` on group execute) — Set Group ID

When set on a directory, new files inherit the directory's group.

```bash
-rwxrwsr-x  shared_dir/          # New files get directory's group
```

```bash
chmod g+s directory/             # Set SGID
chmod 2755 directory/            # Same: 2 + 755
chmod g-s directory/             # Remove SGID
```

### Sticky Bit (`t` on others execute) — Restricted Deletion

When set on a directory, users can only delete files they own.

```bash
drwxrwxrwt  /tmp/                # Anyone can write, but only delete own files
```

```bash
chmod +t directory/              # Set sticky bit
chmod 1777 directory/            # Same: 1 + 777
chmod -t directory/              # Remove sticky bit
```

---

## Numeric with Special Bits

| Value | Special Bit | Example |
|-------|-------------|---------|
| `4000` | SUID | `chmod 4755 file` |
| `2000` | SGID | `chmod 2755 dir` |
| `1000` | Sticky | `chmod 1777 /tmp` |

**Combined**: `chmod 6755 file` = SUID + SGID + 755

---

## Default Permissions and `umask`

When you create a new file or directory, Linux assigns default permissions based on a system setting called **umask**.

### How It Works

Linux starts with these **base** permissions:

| Type | Base Permission |
|------|-----------------|
| File | `666` (`rw-rw-rw-`) |
| Directory | `777` (`rwxrwxrwx`) |

Then it **subtracts** the umask value to get the final permission.

### Common umask Values

| umask | File Result | Directory Result | Typical Use |
|-------|-------------|------------------|-------------|
| `022` | `644` (`rw-r--r--`) | `755` (`rwxr-xr-x`) | Most systems (balanced) |
| `002` | `664` (`rw-rw-r--`) | `775` (`rwxrwxr-x`) | Shared groups (collaborative) |
| `077` | `600` (`rw-------`) | `700` (`rwx------`) | Maximum privacy |
| `027` | `640` (`rw-r-----`) | `750` (`rwxr-x---`) | Secure multi-user systems |

### Checking Your umask

```bash
umask                           # Show current umask (octal)
umask -S                        # Show in symbolic format
```

**Example output:**
```bash
$ umask
0022

$ umask -S
u=rwx,g=rx,o=rx
```

### Changing umask

**Temporarily** (current session only):
```bash
umask 027
```

**Permanently**:

Add to your shell configuration file:

```bash
# For Bash
echo "umask 027" >> ~/.bashrc

# For Zsh
echo "umask 027" >> ~/.zshrc

# System-wide (affects all users)
sudo nano /etc/profile          # Add umask line
sudo nano /etc/login.defs       # Some systems use this
```

> **Files vs Directories**: Files never get execute by default (base is 666), even if umask would allow it. This is a security measure — you must explicitly `chmod +x` a file to make it executable.

---

## Viewing Permissions

### `ls -l` — Detailed Listing

```bash
ls -l file.txt                  # Single file
ls -la                          # All files including hidden
ls -ld directory/               # Directory permissions (not contents)
ls -lR directory/               # Recursive listing
```

### `stat` — Detailed File Info

```bash
stat file.txt                   # Full metadata including permissions
stat -c "%a %n" *               # Show octal permissions + filenames
```

---

## Practical Examples

```bash
# Make a script executable
chmod +x deploy.sh

# Secure a private key
chmod 600 ~/.ssh/id_rsa

# Make a file read-only
chmod 444 important.conf

# Set up a shared directory
mkdir shared
chown :developers shared/
chmod 2775 shared/              # SGID + group writable

# Create a dropbox (others can write, only owner can delete)
mkdir dropbox
chmod 1777 dropbox/             # Sticky bit

# Restore default permissions
chmod 644 file.txt              # Default file
chmod 755 directory/            # Default directory

# Fix SSH directory permissions
chmod 700 ~/.ssh/
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
chmod 644 ~/.ssh/known_hosts
chmod 644 ~/.ssh/config
```

---

## Quick Reference

| Command | Purpose |
|---------|---------|
| `chmod 755 file` | Set exact permissions |
| `chmod u+x file` | Add execute for owner |
| `chmod go-w file` | Remove write for group and others |
| `chmod o=rx file` | Set exact permissions for others |
| `chmod -R 755 dir` | Recursive permission change |
| `chown user:group file` | Change owner and group |
| `chgrp group file` | Change group only |
| `umask` | Show/set default permission mask |
| `ls -l` | View permissions |
| `stat file` | Detailed permission info |

| Octal | Permissions | Common Use |
|-------|-------------|------------|
| `777` | `rwxrwxrwx` | Open access |
| `755` | `rwxr-xr-x` | Scripts, executables, directories |
| `644` | `rw-r--r--` | Regular files |
| `700` | `rwx------` | Private files |
| `750` | `rwxr-x---` | Group-only access |
| `600` | `rw-------` | Sensitive files (SSH keys, etc.) |
| `660` | `rw-rw----` | Shared group files |
| `640` | `rw-r-----` | Owner+group read, owner write |
| `400` | `r--------` | Read-only, immutable |
