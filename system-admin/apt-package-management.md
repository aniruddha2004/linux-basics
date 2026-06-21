# APT Package Management

> Managing software packages on Debian-based systems (Ubuntu, Debian, Mint, etc.) using APT, `apt-get`, and `apt-cache`.

---

## What is APT?

**APT** (Advanced Package Tool) is a package management system used by Debian-based Linux distributions. It handles:

- Downloading and installing software from repositories
- Resolving dependencies automatically
- Updating and upgrading installed packages
- Removing software cleanly

APT works with **.deb** packages and uses online repositories to find and install software.

---

## How APT Works Behind the Scenes

### Configuration Directory: `/etc/apt/`

```bash
/etc/apt/
├── sources.list          # Main repository list
├── sources.list.d/       # Additional repository files (one per source)
├── apt.conf.d/           # APT configuration snippets
├── preferences.d/        # Pinning preferences
├── trusted.gpg.d/        # GPG keys for verifying packages
└── keyrings/             # Modern keyring location
```

### The Package Index

When you run `apt update`, APT downloads package information from all configured repositories into:

```bash
/var/lib/apt/lists/       # Cached package lists
/var/cache/apt/archives/  # Downloaded .deb packages
```

---

## `apt-get` — Core APT Commands

### Update Package Lists

Download the latest package information from repositories.

```bash
sudo apt-get update
```

> **Must run this first** before installing or upgrading packages.

---

### Upgrade Installed Packages

Upgrade all installed packages to their newest available versions.

```bash
sudo apt-get upgrade          # Safe upgrade (won't remove packages)
sudo apt-get dist-upgrade     # Full upgrade (may remove/install packages)
```

| Command | Behavior |
|---------|----------|
| `upgrade` | Upgrades packages, skips if dependencies change |
| `dist-upgrade` | Upgrades packages, handles dependency changes |

---

### Install Packages

```bash
sudo apt-get install package_name
sudo apt-get install package1 package2    # Install multiple
sudo apt-get install -y package_name      # Auto-confirm (no prompt)
sudo apt-get install ./local-package.deb  # Install local .deb file
```

---

### Remove Packages

```bash
sudo apt-get remove package_name          # Remove package (keep config)
sudo apt-get purge package_name           # Remove package + config files
sudo apt-get autoremove                   # Remove unused dependencies
sudo apt-get autoremove --purge           # Remove unused deps + configs
```

| Command | What It Removes |
|---------|----------------|
| `remove` | Package binaries only |
| `purge` | Package + configuration files |
| `autoremove` | Dependencies no longer needed |

---

### Download Without Installing

```bash
sudo apt-get download package_name        # Download .deb to current directory
sudo apt-get source package_name          # Download source code
```

---

### Clean Up

```bash
sudo apt-get clean                        # Clear downloaded packages cache
sudo apt-get autoclean                    # Remove old package versions only
```

---

## `apt-cache` — Query Package Information

### Search for Packages

```bash
apt-cache search nginx                    # Search by name/description
apt-cache search --names-only nginx       # Search package names only
```

---

### Show Package Details

```bash
apt-cache show nginx                      # Full package information
apt-cache showpkg nginx                   # Show dependencies and reverse dependencies
```

---

### Check Package Policy

```bash
apt-cache policy nginx                    # Show versions from different repos
apt-cache policy                          # Show all repository priorities
```

---

### View Dependencies

```bash
apt-cache depends nginx                   # What nginx depends on
apt-cache rdepends nginx                  # What depends on nginx
```

---

### List All Packages

```bash
apt-cache pkgnames                        # List all available package names
apt-cache pkgnames nginx                  # List packages starting with "nginx"
```

---

## Repository Configuration (`/etc/apt/sources.list`)

Each line in `sources.list` follows this format:

```
deb [options] URL distribution component [component2 ...]
```

### Example `sources.list`

```bash
# Ubuntu main repositories
deb http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-updates main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-security main restricted universe multiverse

# Source code repositories (optional)
deb-src http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
```

### Components Explained

| Component | Description |
|-----------|-------------|
| `main` | Officially supported free software |
| `restricted` | Officially supported non-free software (drivers, etc.) |
| `universe` | Community-maintained free software |
| `multiverse` | Non-free software (legal restrictions) |

---

## Adding a Repository

### Method 1: Edit `sources.list` Directly

```bash
sudo nano /etc/apt/sources.list
# Add your repository line, then save

sudo apt-get update
```

### Method 2: Create a Separate File (Recommended)

```bash
sudo nano /etc/apt/sources.list.d/custom-repo.list
# Add repository line to this file

sudo apt-get update
```

> **Best Practice**: Use `sources.list.d/` instead of editing `sources.list` directly. Easier to manage and remove.

### Method 3: Using `add-apt-repository` (Ubuntu/Debian)

```bash
# Add a PPA (Personal Package Archive)
sudo add-apt-repository ppa:user/ppa-name

# Add a generic repository
sudo add-apt-repository "deb http://example.com/repo stable main"

# Add with GPG key
sudo add-apt-repository --keyserver keyserver.ubuntu.com --recv-keys KEY_ID
```

---

## Adding Repository GPG Keys

Repositories are signed with GPG keys to verify package authenticity.

### Method 1: Using `apt-key` (Legacy)

```bash
wget -qO - https://example.com/key.gpg | sudo apt-key add -
sudo apt-key list                           # List added keys
```

> **Deprecated**: `apt-key` is deprecated. Use Method 2 or 3 instead.

### Method 2: Using `gpg` Directly (Modern)

```bash
# Download and add key to trusted keyrings
wget -qO - https://example.com/key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/example-archive-keyring.gpg

# Then reference it in sources.list:
deb [signed-by=/usr/share/keyrings/example-archive-keyring.gpg] https://example.com/repo stable main
```

### Method 3: Using `curl` + `gpg`

```bash
curl -fsSL https://example.com/key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/example-keyring.gpg
```

---

## Complete Example: Adding Docker Repository

```bash
# 1. Install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

# 2. Create keyrings directory
sudo install -m 0755 -d /etc/apt/keyrings

# 3. Download and add GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 4. Add repository to sources.list.d
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. Update package index
sudo apt-get update

# 6. Install package
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

---

## Quick Reference

| Command | Purpose |
|---------|---------|
| `apt-get update` | Refresh package lists |
| `apt-get upgrade` | Upgrade installed packages |
| `apt-get dist-upgrade` | Full upgrade with dependency resolution |
| `apt-get install pkg` | Install a package |
| `apt-get remove pkg` | Remove a package |
| `apt-get purge pkg` | Remove package + config |
| `apt-get autoremove` | Remove unused dependencies |
| `apt-get clean` | Clear package cache |
| `apt-cache search term` | Search for packages |
| `apt-cache show pkg` | Show package info |
| `apt-cache policy pkg` | Show version/policy info |
| `apt-cache depends pkg` | Show dependencies |
| `apt-cache rdepends pkg` | Show reverse dependencies |

---

## Important Files

| File/Directory | Purpose |
|----------------|---------|
| `/etc/apt/sources.list` | Main repository configuration |
| `/etc/apt/sources.list.d/` | Additional repository files |
| `/etc/apt/trusted.gpg.d/` | Trusted GPG keys |
| `/etc/apt/keyrings/` | Modern keyring location |
| `/var/lib/apt/lists/` | Downloaded package lists |
| `/var/cache/apt/archives/` | Cached .deb packages |
| `/var/log/apt/history.log` | APT operation history |

---

## Tips

- **Always run `apt update` before installing or upgrading**
- Use `apt-get` in scripts; `apt` (newer) is fine for interactive use
- Use `purge` instead of `remove` to completely clean up a package
- Run `autoremove` periodically to clean unused dependencies
- Pin packages in `/etc/apt/preferences` if you need to hold specific versions
