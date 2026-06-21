# Basic Linux Commands

> Essential commands for everyday Linux navigation and file management.

---

## Table of Contents

1. [Navigation](#navigation)
2. [File Management](#file-management)
3. [Directory Operations](#directory-operations)
4. [System Information](#system-information)
5. [Help & Manuals](#help--manuals)

---

## Navigation

### `pwd` — Print Working Directory
Shows the full path of the current directory.

```bash
pwd
```

| Flag | Description |
|------|-------------|
| `-P` | Print physical path (resolving all symlinks) |
| `-L` | Print logical path (with symlinks) |

---

### `ls` — List Directory Contents
Lists files and directories in the current or specified directory.

```bash
ls                  # List contents of current directory
ls /var/log         # List contents of specific directory
```

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-a` | `--all` | Show hidden files (starting with `.`) |
| `-l` | | Long listing format (permissions, size, date, etc.) |
| `-h` | `--human-readable` | Human-readable file sizes (KB, MB, GB) |
| `-t` | | Sort by modification time (newest first) |
| `-r` | `--reverse` | Reverse order of sorting |
| `-S` | | Sort by file size (largest first) |
| `-R` | `--recursive` | List subdirectories recursively |
| `-i` | | Show inode number |
| `-d` | `--directory` | List directory itself, not its contents |

**List only directories:**
```bash
ls -d */              # List only directories in current location
```

**Common combos:**
```bash
ls -la              # Show all files in long format
ls -lh              # Human-readable sizes in long format
ls -ltr             # Sort by time, reversed (oldest last)
ls -lha             # All files + human-readable + long format
```

---

### `cd` — Change Directory
Move between directories.

```bash
cd /home/user        # Go to specific directory
cd ..               # Go up one directory
cd ~                # Go to home directory
cd -                # Go to previous directory
cd /                # Go to root directory
cd ../..            # Go up two directories
```

---

## File Management

### `touch` — Create Empty Files / Update Timestamps
Creates an empty file or updates the access/modification time of an existing file. (Use `stat` to view atime/mtime.)

```bash
touch file.txt                    # Create empty file
touch file1.txt file2.txt         # Create multiple files
touch -t 202312011200 file.txt   # Set specific timestamp
```

| Flag | Description |
|------|-------------|
| `-a` | Change access time only |
| `-m` | Change modification time only |
| `-t` | Use specified time instead of current time |
| `-c` | Do not create any files |

---

### `cat` — Concatenate and Display Files
Displays the contents of files. Often used to view small files.

```bash
cat file.txt                      # Display file contents
cat file1.txt file2.txt           # Concatenate and display multiple files
cat -n file.txt                   # Display with line numbers
cat > newfile.txt                 # Create/overwrite file (Ctrl+D to save)
cat >> file.txt                   # Append to file (Ctrl+D to save)
cat file1.txt file2.txt > combined.txt  # Combine files into one
cat file1.txt >> existing.txt     # Append contents to existing file
```

| Flag | Description |
|------|-------------|
| `-n` | Number all output lines |
| `-b` | Number non-empty output lines |
| `-s` | Suppress repeated empty lines |
| `-E` | Display `$` at end of each line |
| `-T` | Display `^I` for tabs |
| `>` | Redirect output to file (overwrite) |
| `>>` | Redirect output to file (append) |

---

### `less` — View Files Page by Page
View large files one screen at a time (better than `cat` for big files).

```bash
less /var/log/syslog
less -N file.txt                  # Show line numbers
```

| Flag | Description |
|------|-------------|
| `-N` | Show line numbers |
| `-S` | Chop long lines (no wrapping) |
| `-i` | Case-insensitive search (lowercase only) |
| `-I` | Case-insensitive search (always, including uppercase) |
| `+/pattern` | Start at first occurrence of pattern (forward) |
| `?/pattern` | Start at first occurrence of pattern (backward) |

> **Note on search patterns**: `+/pattern` opens the file at the first forward match; `?/pattern` opens at the first backward match.

**Navigation in `less`:**
- `Space` or `Page Down` — Next page
- `b` or `Page Up` — Previous page
- `q` — Quit
- `/pattern` — Search forward
- `?pattern` — Search backward
- `n` — Next match
- `N` — Previous match
- `g` — Go to beginning
- `G` — Go to end

---

### `more` — View Files Page by Page (Simpler)
Basic pager, less interactive than `less`.

```bash
more file.txt
```

---

### `head` — Show Beginning of File
Display the first few lines of a file.

```bash
head file.txt                     # First 10 lines (default)
head -n 5 file.txt                # First 5 lines
head -n -5 file.txt               # All except last 5 lines
head -c 100 file.txt              # First 100 bytes
```

---

### `tail` — Show End of File
Display the last few lines of a file.

```bash
tail file.txt                     # Last 10 lines (default)
tail -n 5 file.txt                # Last 5 lines
tail -n +5 file.txt               # From line 5 to end
tail -c 100 file.txt              # Last 100 bytes
```

| Flag | Description |
|------|-------------|
| `-f` | Follow file as it grows (watch live logs) |
| `-F` | Like `-f`, but also retry if file is inaccessible |

```bash
tail -f /var/log/syslog           # Watch log file in real-time
```

---

### `wc` — Word Count
Count lines, words, and bytes in files.

```bash
wc file.txt                       # Lines, words, bytes
wc -l file.txt                    # Count lines only
wc -w file.txt                    # Count words only
wc -c file.txt                    # Count bytes only
wc -m file.txt                    # Count characters
ls | wc -l                        # Count number of files
```

| Flag | Description |
|------|-------------|
| `-l` | Count lines |
| `-w` | Count words |
| `-c` | Count bytes |
| `-m` | Count characters |

---

### `cut` — Extract Columns/Fields
Extract specific columns or fields from lines.

```bash
cut -d':' -f1 /etc/passwd         # Extract first field (username)
cut -d':' -f1,3 /etc/passwd       # Extract fields 1 and 3
cut -c1-5 file.txt                # Extract characters 1 to 5
cut -c1,3,5 file.txt              # Extract characters 1, 3, and 5
ls -l | cut -c1-10                # Extract first 10 characters
```

| Flag | Description |
|------|-------------|
| `-d` | Delimiter (default: tab) |
| `-f` | Fields to extract |
| `-c` | Characters to extract |
| `--complement` | Extract everything except specified fields |

---

### `sort` — Sort Lines
Sort lines in a file alphabetically or numerically.

```bash
sort file.txt                     # Sort alphabetically
sort -r file.txt                  # Reverse sort
sort -n file.txt                  # Numeric sort
sort -u file.txt                  # Sort and remove duplicates
sort -k2 file.txt                 # Sort by second field
sort -t',' -k2n file.txt         # Sort by 2nd column, numeric, comma-delimited
cat file.txt | sort | uniq       # Sort and show unique lines
```

| Flag | Description |
|------|-------------|
| `-r` | Reverse order |
| `-n` | Numeric sort |
| `-u` | Unique (remove duplicates) |
| `-k` | Sort by key/column |
| `-t` | Field separator |
| `-f` | Ignore case |
| `-M` | Month sort |

---

### `grep` — Global Regular Expression Print
Search for patterns in text. One of the most powerful and frequently used commands.

```bash
grep "pattern" file.txt           # Search for pattern in file
grep -i "pattern" file.txt        # Case-insensitive search
grep -v "pattern" file.txt        # Invert match (lines NOT matching)
grep -n "pattern" file.txt        # Show line numbers
grep -c "pattern" file.txt        # Count matching lines
grep -l "pattern" *.txt           # List files that contain pattern
grep -r "pattern" dir/            # Recursively search directory
grep -w "word" file.txt           # Match whole word only
grep -A 2 "pattern" file.txt      # Show 2 lines after match
grep -B 2 "pattern" file.txt      # Show 2 lines before match
grep -C 2 "pattern" file.txt      # Show 2 lines before and after
grep -E "pat1|pat2" file.txt      # Extended regex (OR pattern)
```

| Flag | Description |
|------|-------------|
| `-i` | Ignore case |
| `-v` | Invert match (exclude) |
| `-n` | Show line numbers |
| `-c` | Count matches |
| `-l` | List filenames only |
| `-r` or `-R` | Recursive search |
| `-w` | Match whole words only |
| `-x` | Match whole lines only |
| `-A N` | Show N lines after match |
| `-B N` | Show N lines before match |
| `-C N` | Show N lines before and after |
| `-E` | Extended regular expressions |
| `-F` | Fixed strings (no regex) |
| `-o` | Show only matching part |
| `--color=auto` | Highlight matches |

**Common Combos:**

```bash
grep -in "error" log.txt          # Case-insensitive, with line numbers
grep -rl "TODO" src/              # Find files containing TODO
grep -Ev "^#|^$" file.txt        # Remove comments and blank lines
grep -P "\d{3}-\d{4}" file.txt   # Perl-compatible regex
history | grep "git commit"       # Search command history
ps aux | grep firefox             # Find running process
```

**Regular Expressions Basics:**

```bash
grep "^hello" file.txt            # Lines starting with "hello"
grep "world$" file.txt            # Lines ending with "world"
grep "a.c" file.txt               # "a", any character, "c"
grep "a*c" file.txt               # Zero or more "a" followed by "c"
grep "[aeiou]" file.txt           # Any vowel
grep "[^0-9]" file.txt            # Any non-digit
grep "\.txt$" file.txt            # Files ending with .txt
```

> **Tip**: `grep` uses **basic regex** by default. Use `-E` for extended regex (supports `|`, `+`, `?`, `()` without escaping) or `-P` for Perl-compatible regex.

---

### `cp` — Copy Files and Directories
Copy files or directories.

```bash
cp file.txt backup.txt            # Copy file
cp file1.txt file2.txt /backup/   # Copy multiple files to directory
cp -r dir1/ dir2/                 # Copy directory recursively
cp -i file.txt dest.txt           # Prompt before overwriting
```

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-r` or `-R` | `--recursive` | Copy directories recursively |
| `-i` | `--interactive` | Prompt before overwriting |
| `-v` | `--verbose` | Show what's being copied |
| `-p` | `--preserve` | Preserve attributes (mode, ownership, timestamps) |
| `-f` | `--force` | Force overwrite without prompting |
| `-u` | `--update` | Copy only when source is newer |
| `-a` | `--archive` | Archive mode (same as `-dR --preserve=all`) |

---

### `mv` — Move or Rename Files
Move files/directories or rename them.

```bash
mv file.txt newname.txt           # Rename file
mv file.txt /destination/         # Move file to directory
mv *.txt /backup/                 # Move all .txt files
mv -i file.txt dest.txt           # Prompt before overwriting
```

| Flag | Description |
|------|-------------|
| `-i` | Interactive (prompt before overwrite) |
| `-v` | Verbose |
| `-f` | Force (no prompt) |
| `-n` | No clobber (don't overwrite) |

---

### `rm` — Remove Files and Directories
Delete files or directories permanently.

```bash
rm file.txt                       # Remove file
rm file1.txt file2.txt            # Remove multiple files
rm -r directory/                  # Remove directory and contents
rm -rf directory/                 # Force remove recursively
rm -i file.txt                    # Prompt before removing
```

| Flag | Description |
|------|-------------|
| `-r` or `-R` | Recursive (for directories) |
| `-f` | Force (no prompt, ignore errors) |
| `-i` | Interactive (prompt before removal) |
| `-v` | Verbose (show what's being removed) |
| `-d` | Remove empty directories |

⚠️ **Warning**: `rm -rf /` can destroy your system. Always double-check paths!

---

## Directory Operations

### `mkdir` — Make Directories
Create new directories.

```bash
mkdir newdir                      # Create single directory
mkdir dir1 dir2 dir3              # Create multiple directories
mkdir -p path/to/new/dir          # Create parent directories as needed
```

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-p` | `--parents` | Create parent directories if they don't exist |
| `-v` | `--verbose` | Show each created directory |
| `-m` | `--mode` | Set permissions (e.g., `-m 755`) |

---

### `rmdir` — Remove Empty Directories
Remove empty directories only.

```bash
rmdir emptydir                    # Remove empty directory
rmdir dir1 dir2                   # Remove multiple empty directories
rmdir -p path/to/dir              # Remove directory and its empty parents
```

| Flag | Description |
|------|-------------|
| `-p` | Remove parent directories if they become empty |
| `-v` | Verbose |

> **Note**: For non-empty directories, use `rm -r` or `rm -rf`.

---

## System Information

### `whoami` — Show Current Username

```bash
whoami
```

---

### `who` — Show Logged-in Users

```bash
who
who -H                            # Show column headers
who -u                            # Show idle time and process ID
```

---

### `uname` — Print System Information

```bash
uname                             # Kernel name
uname -a                          # All system info
uname -r                          # Kernel release/version
uname -m                          # Machine hardware name
uname -n                          # Network node hostname
```

| Flag | Description |
|------|-------------|
| `-a` | All information |
| `-s` | Kernel name |
| `-n` | Network node hostname |
| `-r` | Kernel release |
| `-v` | Kernel version |
| `-m` | Machine hardware name |
| `-p` | Processor type |
| `-o` | Operating system |

---

### `hostname` — Show or Set System Hostname

```bash
hostname                          # Show current hostname
hostname -I                       # Show IP addresses
```

---

### `clear` — Clear Terminal Screen

```bash
clear
```

Or press `Ctrl + L` as a shortcut.

---

### `date` — Display or Set Date and Time

```bash
date                              # Current date and time
date +"%Y-%m-%d %H:%M:%S"        # Custom format
date -u                           # UTC time
```

| Flag | Description |
|------|-------------|
| `-u` | Display time in UTC |
| `+FORMAT` | Custom format string |

**Common format specifiers:**
- `%Y` — Year (e.g., 2024)
- `%m` — Month (01-12)
- `%d` — Day (01-31)
- `%H` — Hour (00-23)
- `%M` — Minute (00-59)
- `%S` — Second (00-59)

---

### `uptime` — Show System Uptime

```bash
uptime                            # Show uptime, users, load average
uptime -p                         # Pretty format (human-readable)
```

---

## Help & Manuals

### `man` — Manual Pages
View detailed documentation for commands.

```bash
man ls                            # Show manual for ls
man 5 passwd                      # Section 5 manual for passwd file
man -k keyword                    # Search for keyword in manuals
```

| Flag | Description |
|------|-------------|
| `-k` | Search for keyword |
| `-f` | Equivalent to `whatis` |
| `-a` | Show all matching pages |

---

### `info` — Info Pages
Alternative documentation system (often more detailed than man).

```bash
info ls
info coreutils                    # Info for GNU core utilities
```

---

### `whatis` — Brief Command Description
One-line summary of a command.

```bash
whatis ls
whatis -r "list"                  # Regex search
```

---

### `which` — Locate a Command
Find the path of an executable.

```bash
which ls
which python3
which -a python                   # Show all occurrences
```

---

### `type` — Describe a Command
Show how a command is interpreted by the shell.

```bash
type ls                           # Show if ls is built-in or external
type cd                           # Show if cd is a shell built-in
type -a ls                        # Show all locations
```

---

### `alias` — Create Command Shortcuts

```bash
alias                           # Show all aliases
alias ll='ls -la'               # Create alias
alias grep='grep --color=auto'  # Create alias with color
```

> To make aliases permanent, add them to `~/.bashrc` or `~/.zshrc`.

---

## Quick Reference Table

| Command | Purpose |
|---------|---------|
| `pwd` | Show current directory |
| `ls` | List files |
| `cd` | Change directory |
| `touch` | Create empty file |
| `cat` | Display file contents |
| `less` | View files interactively |
| `head` | Show first lines |
| `tail` | Show last lines |
| `wc` | Count lines, words, bytes |
| `cut` | Extract columns/fields |
| `sort` | Sort lines |
| `grep` | Search text patterns |
| `cp` | Copy files/directories |
| `mv` | Move/rename files |
| `rm` | Remove files/directories |
| `mkdir` | Create directories |
| `rmdir` | Remove empty directories |
| `whoami` | Show username |
| `who` | Show logged-in users |
| `uname` | System information |
| `date` | Show/set date and time |
| `man` | Manual pages |
| `which` | Locate commands |
| `alias` | Command shortcuts |
