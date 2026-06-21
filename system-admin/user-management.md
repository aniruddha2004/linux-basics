# User Account Management

> Basics of managing user accounts on Linux: the files that store user data and the commands to create, modify, and delete users.

---

## The Three Core User Files

Linux stores user information in three main files under `/etc/`:

| File | Purpose | Permissions |
|------|---------|-------------|
| `/etc/passwd` | User account information (username, UID, GID, home, shell) | Readable by all |
| `/etc/shadow` | Encrypted passwords and password aging | Readable by root only |
| `/etc/group` | Group definitions and member lists | Readable by all |

---

## `/etc/passwd` — User Account Information

Each line represents one user account, with fields separated by colons (`:`).

### Format

```
username:password:UID:GID:GECOS:home_directory:shell
```

### Example from your system

```
aniruddha_ghosh:x:1000:1000:,,,:/home/aniruddha_ghosh:/usr/bin/bash
```

| Field | Value | Meaning |
|-------|-------|---------|
| `username` | `aniruddha_ghosh` | Login name |
| `password` | `x` | Placeholder — actual password is in `/etc/shadow` |
| `UID` | `1000` | User ID. `0` = root, `1-999` = system users, `1000+` = regular users |
| `GID` | `1000` | Primary Group ID |
| `GECOS` | `,,,` | Comment/Full name (can include name, phone, etc.) |
| `home_directory` | `/home/aniruddha_ghosh` | Path to user's home folder |
| `shell` | `/usr/bin/bash` | Default shell when user logs in |

### System User Example

```
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
```

| Field | Value | Meaning |
|-------|-------|---------|
| `username` | `systemd-network` | Service account for systemd networking |
| `UID` | `998` | System user (< 1000) |
| `shell` | `/usr/sbin/nologin` | Cannot log in interactively |

> **System users** (UID 1–999) run services and daemons. They typically have `/usr/sbin/nologin` or `/bin/false` as their shell to prevent interactive login.

---

## `/etc/shadow` — Encrypted Passwords

Stores actual password hashes and password aging policies. Only readable by root.

### Format

```
username:password_hash:last_change:min_age:max_age:warn_period:inactive_period:expiration_date:reserved
```

### Example from your system

```
aniruddha_ghosh:$y$j9T$j8y2TAjbgbC9XVLZf2H8P/$R6Zi6PvRY5bR08HOiMR5MkqnRa86lWrHBRfMLpMBPv7:20445:0:99999:7:::
```

| Field | Value | Meaning |
|-------|-------|---------|
| `username` | `aniruddha_ghosh` | Matches entry in `/etc/passwd` |
| `password_hash` | `$y$j9T$...` | Encrypted password hash (algorithm varies) |
| `last_change` | `20445` | Days since Jan 1, 1970 when password was last changed |
| `min_age` | `0` | Minimum days before password can be changed |
| `max_age` | `99999` | Maximum days before password must be changed |
| `warn_period` | `7` | Days before expiry to warn user |
| `inactive_period` | (empty) | Days after expiry before account is disabled |
| `expiration_date` | (empty) | Account expiration date (days since epoch) |
| `reserved` | (empty) | Reserved for future use |

### Special Password Values

| Symbol | Meaning |
|--------|---------|
| `*` | No password set, account locked |
| `!` | Password disabled (locked) |
| `!$hash` | Locked but hash preserved (unlock restores it) |
| Empty | No password required to log in |

```bash
# Locked account (password prefixed with !)
test-user:!$y$j9T$...:20625:0:99999:7:::

# Unlocked account
test-user:$y$j9T$...:20625:0:99999:7:::

# No password (system accounts)
root:*:20305:0:99999:7:::
```

---

## `/etc/group` — Group Definitions

Defines groups and their member lists.

### Format

```
group_name:password:GID:member_list
```

### Example from your system

```
sudo:x:27:aniruddha_ghosh
docker:x:109:
ollama:x:989:aniruddha_ghosh
```

| Field | Example | Meaning |
|-------|---------|---------|
| `group_name` | `sudo` | Name of the group |
| `password` | `x` | Placeholder (rarely used) |
| `GID` | `27` | Group ID |
| `member_list` | `aniruddha_ghosh` | Comma-separated list of users in this group |

> **Primary Group**: Stored in `/etc/passwd` (GID field). When you create a file, it belongs to this group by default.<br>
> **Secondary Groups**: Listed in `/etc/group`. You can belong to many secondary groups.

---

## Creating a User: `useradd`

Low-level command for creating user accounts.

```bash
sudo useradd -m -d /home/test-user -s /bin/bash test-user
```

| Flag | Description |
|------|-------------|
| `-m` | Create home directory |
| `-d /path` | Specify home directory path |
| `-s /shell` | Specify default shell |
| `-u UID` | Set specific User ID |
| `-g GID` | Set primary group |
| `-G group1,group2` | Add to secondary groups |
| `-c "Comment"` | Add GECOS comment |
| `-r` | Create a system user (UID < 1000) |

What happens when you run `useradd -m`:

1. Entry added to `/etc/passwd`
2. Entry added to `/etc/shadow` (with `!` or `*` — no password yet)
3. Entry added to `/etc/group` (creates a group with same name)
4. Home directory created at `/home/username`
5. Files from `/etc/skel/` are copied to the home directory

---

## `/etc/skel` — Skeleton Directory

Contains template files copied to every new user's home directory.

```bash
ls -a /etc/skel/
.bash_logout  .bashrc  .profile
```

These are default shell configuration files that give new users a working environment immediately.

---

## Setting a Password: `passwd`

```bash
sudo passwd username          # Set/change password as root
passwd                        # Change your own password
```

Running this updates the password hash in `/etc/shadow`.

---

## Modifying a User: `usermod`

```bash
sudo usermod -L username      # Lock account (prepend ! to password)
sudo usermod -U username      # Unlock account (remove !)
sudo usermod -l newname oldname  # Change username
sudo usermod -d /new/home -m username  # Move home directory
sudo usermod -s /bin/zsh username      # Change default shell
sudo usermod -aG docker username       # Append to secondary group
```

| Flag | Description |
|------|-------------|
| `-L` | Lock password |
| `-U` | Unlock password |
| `-l` | Change login name |
| `-d` | Change home directory (`-m` moves files) |
| `-s` | Change shell |
| `-aG` | Append to group(s) |
| `-g` | Change primary group |

---

## Deleting a User: `userdel`

```bash
sudo userdel username         # Delete user (keeps home directory)
sudo userdel -r username      # Delete user AND home directory
```

> **Note**: `userdel` without `-r` leaves the home directory behind. You may need to manually remove `/home/username` afterward.

---

## `useradd` vs `adduser`

| | `useradd` | `adduser` |
|--|-----------|-----------|
| **Type** | Low-level binary | Higher-level Perl script (Debian/Ubuntu) |
| **Interaction** | Silent, needs all flags | Interactive, asks questions |
| **Home directory** | Not created by default (need `-m`) | Created automatically |
| **Password** | Not set (locked) | Prompts to set password |
| **Skeleton files** | Copied with `-m` | Copied automatically |
| **Use in scripts** | Preferred | Not ideal |
| **Interactive use** | Verbose | User-friendly |

```bash
# useradd — manual, requires flags
sudo useradd -m -s /bin/bash johnsudo passwd john

# adduser — interactive, guides you through
sudo adduser john
# (asks for password, full name, room number, phone, etc.)
```

> **Recommendation**: Use `adduser` interactively on Debian/Ubuntu systems. Use `useradd` in scripts or when you need precise control.

---

## Quick Reference

| Task | Command |
|------|---------|
| Create user (with home) | `sudo useradd -m -s /bin/bash username` |
| Create user (interactive) | `sudo adduser username` |
| Set password | `sudo passwd username` |
| Lock account | `sudo usermod -L username` |
| Unlock account | `sudo usermod -U username` |
| Add to group | `sudo usermod -aG groupname username` |
| Change shell | `sudo usermod -s /bin/zsh username` |
| Delete user (keep home) | `sudo userdel username` |
| Delete user (remove home) | `sudo userdel -r username` |
| View user info | `id username` |
| View all users | `cat /etc/passwd` |
| View all groups | `cat /etc/group` |

| File | Contains |
|------|----------|
| `/etc/passwd` | Username, UID, GID, home, shell |
| `/etc/shadow` | Password hash, aging info |
| `/etc/group` | Group names, GIDs, members |
| `/etc/skel` | Template files for new users |
