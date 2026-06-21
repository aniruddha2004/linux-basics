# Process Management Commands

> Practical commands for inspecting, controlling, and adjusting running processes.

---

## `ps` — Process Status

Display information about active processes. `ps` has multiple syntax styles — Unix (`-`), BSD (no dash), and GNU (`--`).

### Common Flags

| Flag | Style | Meaning |
|------|-------|---------|
| `a` | BSD | Show processes for **all users** that are attached to a terminal (excludes session leaders) |
| `u` | BSD | User-oriented format (shows CPU%, MEM%, command args) |
| `x` | BSD | Include processes **not attached to a terminal** (daemons, background jobs) |
| `e` | Unix | Show **every process** on the system (equivalent to `-A`) |
| `f` | Unix | Full format listing with ASCII art process tree |
| `L` | Both | Show threads (LWPs) — one line per thread |

### Common Combinations

```bash
ps                               # Processes attached to current terminal
ps -f                            # Full format for current terminal
ps -a                            # All users' terminal-attached processes
ps -e                            # Every process on the system
ps -ef                           # Every process, full format
ps aux                           # All processes (BSD style), detailed
ps -af                           # All users, full format, terminal only
ps -afu                          # All users, full+user format, terminal only
ps -afux                         # All users, full+user format, including daemons
ps aux | grep firefox           # Find a specific process
ps -fL -C firefox               # Show threads of a specific command
ps -efL | grep opencode         # Show all threads of a process (from theory)
```

### Output Examples

#### `ps` — Default (current terminal only)

```bash
$ ps
    PID TTY          TIME CMD
  65513 pts/7    00:00:00 bash
  76424 pts/7    00:00:00 ps
```

> Only shows processes attached to your current terminal (bash shell + ps itself).

---

#### `ps -f` — Full format, current terminal

```bash
$ ps -f
UID          PID    PPID  C STIME TTY          TIME CMD
anirudd+   65513   65511  0 Jun20 pts/7    00:00:00 -bash
anirudd+   76425   65513  0 13:43 pts/7    00:00:00 ps -f
```

> Adds UID, PPID, C (CPU usage), STIME. Same scope as `ps` but more columns.

---

#### `ps -a` — All users, terminal-attached only

```bash
$ ps -a
    PID TTY          TIME CMD
    791 pts/1    00:00:00 bash
  72095 pts/0    00:00:21 htop
  72131 pts/2    00:12:15 opencode
  73006 pts/3    00:01:56 opencode
  76427 pts/7    00:00:00 ps
```

> Includes all users' processes **attached to terminals** (htop, opencode, bash shells). Excludes background daemons like systemd, dockerd, rsyslogd.

---

#### `ps -e` — Every process on the system

```bash
$ ps -e
    PID TTY          TIME CMD
      1 ?        00:00:10 systemd
      2 ?        00:00:00 init-systemd(Ub
      6 ?        00:00:01 init
    177 ?        00:00:01 cron
    178 ?        00:00:01 dbus-daemon
    182 ?        00:00:27 ollama
    247 ?        00:10:56 containerd
    377 ?        00:01:18 dockerd
  65513 pts/7    00:00:00 bash
  72131 pts/2    00:12:15 opencode
  ...
```

> **Every process** — includes background daemons, system services, and terminal processes. Notice `?` in TTY column for daemons with no terminal.

---

#### `ps -ef` — Every process, full format

```bash
$ ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 Jun17 ?        00:00:10 /usr/lib/systemd/systemd --system --deserialize=94
root           2       1  0 Jun17 ?        00:00:00 /init
root         377       1  0 Jun17 ?        00:01:18 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
anirudd+   65513   65511  0 Jun20 pts/7    00:00:00 -bash
anirudd+   72131   71044 18 12:38 pts/2    00:12:15 opencode
...
```

> Full format (`-f`) + every process (`-e`). Shows PPID, full command line with arguments. Most commonly used combination.

---

#### `ps -af` — All users, full format, terminal only

```bash
$ ps -af
UID          PID    PPID  C STIME TTY          TIME CMD
anirudd+     791     703  0 Jun17 pts/1    00:00:00 -bash
anirudd+   72095   67683  0 12:30 pts/0    00:00:21 htop
anirudd+   72131   71044 18 12:38 pts/2    00:12:15 opencode
anirudd+   73006   72993 18 13:32 pts/3    00:01:57 opencode
anirudd+   76429   65513  0 13:43 pts/7    00:00:00 ps -af
```

> Like `ps -a` but with full format columns (PPID, C, STIME). Still excludes background daemons.

---

#### `ps -afu` — All users, full+user format, terminal only

```bash
$ ps -afu
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
anirudd+   72993  0.0  0.0   6272  5376 pts/3    Ss   13:32   0:00 -bash
anirudd+   73006 17.6 10.1 76205668 812880 pts/3 Sl+  13:32   1:58  \_ opencode
anirudd+   71044  0.0  0.0   6404  5504 pts/2    Ss   11:01   0:00 -bash
anirudd+   72131 18.6 11.0 76305476 884408 pts/2 Sl+  12:38  12:16  \_ opencode
...
```

> Adds user-oriented columns: `%CPU`, `%MEM`, `VSZ`, `RSS`, `STAT`. Still terminal-only (`-a`). Notice `\_` indicating child processes indented under parents.

---

#### `ps -afux` — All users, full+user, including daemons

```bash
$ ps -afux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.1  22292 11392 ?        Ss   Jun17   0:10 /usr/lib/systemd/systemd --system --deserialize=94
root           2  0.0  0.0   3120  1920 ?        Sl   Jun17   0:00 /init
  root         6  0.0  0.0   3172  2180 ?        Sl   Jun17   0:01  \_ plan9 --control-socket ...
  root       703  0.0  0.0   6696  4352 pts/1    Ss   Jun17   0:00  \_ /bin/login -f
  anirudd+   791  0.0  0.0   6200  4352 pts/1    S+   Jun17   0:00  |   \_ -bash
...
```

> `x` includes daemons (notice `?` for TTY). Tree view shows parent-child relationships with `\_`, `|`, indentation.

---

#### `ps aux` — BSD style, all processes, detailed

```bash
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  22292 11392 ?        Ss   Jun17   0:10 systemd
root       247  0.1  0.5 2097980 47616 ?       Ssl  Jun17  10:56 containerd
anirudd+ 72131 18.5 11.0 76305476 884408 pts/2 Sl+  12:38  12:17 opencode
```

> BSD style (no dash or dash before single letters). Combines `a` (all users), `u` (user format), `x` (no terminal). Same output as `ps -ef` but different column names and order.

---

### `ps` Flag Comparison

| Command | Shows | Includes Daemons | Full Format | User Columns | Tree |
|---------|-------|------------------|-------------|--------------|------|
| `ps` | Your terminal only | No | No | No | No |
| `ps -f` | Your terminal only | No | Yes | No | No |
| `ps -a` | All terminal processes | No | No | No | No |
| `ps -e` | **Every process** | Yes | No | No | No |
| `ps -ef` | **Every process** | Yes | Yes | No | No |
| `ps -af` | All terminal processes | No | Yes | No | No |
| `ps -afu` | All terminal processes | No | Yes | Yes | Yes |
| `ps -afux` | **Every process** | Yes | Yes | Yes | Yes |
| `ps aux` | **Every process** | Yes | Yes | Yes | No |

> **Key insight**: `-e` means "every process" (system-wide). `-a` means "all terminal-attached processes" (excludes background daemons). The `x` flag is what pulls in daemons when using BSD-style flags.

---

### `ps -efL` — Thread-Aware Output

| Column | Meaning |
|--------|---------|
| `USER` | Username running the process |
| `PID` | Process ID |
| `%CPU` | CPU usage percentage |
| `%MEM` | Memory usage percentage |
| `VSZ` | Virtual memory size (KB) |
| `RSS` | Resident set size — physical RAM used (KB) |
| `TTY` | Controlling terminal |
| `STAT` | Process state (R=Running, S=Sleeping, Z=Zombie, T=Stopped) |
| `START` | Start time |
| `TIME` | Cumulative CPU time |
| `COMMAND` | Command that started the process |

### `ps -efL` — Thread-Aware Output

```bash
ps -efL | grep opencode
```

| Column | Meaning |
|--------|---------|
| `UID` | User ID |
| `PID` | Process ID (same for all threads in a process) |
| `PPID` | Parent Process ID |
| `LWP` | LightWeight Process ID — the actual thread ID |
| `C` | CPU utilization |
| `NLWP` | Number of LWPs (threads) in this process |
| `STIME` | Start time |
| `TTY` | Terminal |
| `TIME` | CPU time |
| `CMD` | Command |

> **Remember from theory**: In `ps -efL`, `PID` stays constant across all threads of a process, while `LWP` changes per thread. When `PID == LWP`, that's the main thread.

---

## `kill` — Send Signals to a Process

Send a signal to a process by its PID.

```bash
kill PID                        # Send SIGTERM (15) — graceful shutdown
kill -9 PID                     # Send SIGKILL (9) — force kill
kill -15 PID                    # Same as default: SIGTERM
kill -1 PID                     # Send SIGHUP (1) — reload config
kill -19 PID                    # Send SIGSTOP (19) — pause
kill -18 PID                    # Send SIGCONT (18) — resume
```

| Signal | Number | When to Use |
|--------|--------|-------------|
| `SIGTERM` | `15` | Default. Graceful shutdown. Try this first. |
| `SIGKILL` | `9` | Force kill. No cleanup. Last resort. |
| `SIGHUP` | `1` | Reload configuration without restarting |
| `SIGSTOP` | `19` | Pause process (cannot be caught) |
| `SIGCONT` | `18` | Resume a stopped process |

> **See** `process-theory.md` **for detailed signal theory.**

---

## `pkill` — Kill Processes by Name

Kill processes matching a pattern instead of knowing the PID.

```bash
pkill firefox                   # Kill all firefox processes
pkill -9 firefox                # Force kill all firefox processes
pkill -f "python3 app.py"       # Match full command line
pkill -u username               # Kill all processes owned by user
pkill -n process_name           # Kill only the newest matching process
pkill -o process_name           # Kill only the oldest matching process
```

| Flag | Description |
|------|-------------|
| `-f` | Match full command line (not just process name) |
| `-u` | Match processes owned by specific user |
| `-n` | Only the newest matching process |
| `-o` | Only the oldest matching process |
| `-signal` | Send specific signal (same numbers as `kill`) |

### `kill` vs `pkill`

| | `kill` | `pkill` |
|--|--------|---------|
| **Target** | Specific PID | Pattern matching (name, user, etc.) |
| **Input** | Number (PID) | Text pattern |
| **Scope** | One process at a time | Can match multiple |
| **Use case** | You know the PID | You know the name |

```bash
# Using kill — need PID first
ps aux | grep firefox          # Find PID (e.g., 5678)
kill 5678                       # Kill it

# Using pkill — direct
pkill firefox                   # No need to look up PID
```

---

## `pgrep` — Find PIDs by Name

The companion to `pkill` — find PIDs without killing.

```bash
pgrep firefox                   # Print PID of firefox
pgrep -l firefox                # Print PID + process name
pgrep -a firefox                # Print PID + full command line
pgrep -u username               # Processes owned by user
pgrep -c firefox                # Count matching processes
```

---

## `nice` — Start with Adjusted Priority

Launch a process with a modified nice value (priority).

```bash
nice -n 10 ./script.sh          # Start with nice value +10 (lower priority)
nice -n -5 ./heavy_job          # Start with nice value -5 (higher priority — needs root)
```

| Nice Value | Priority Effect |
|------------|-----------------|
| `-20` | Highest priority (selfish, most CPU time) |
| `0` | Default priority |
| `+19` | Lowest priority (nicest, yields to others) |

> **Root required** for negative nice values (raising priority). Regular users can only increase niceness (lower priority, `+1` to `+19`).

---

## `renice` — Change Priority of Running Process

Adjust the nice value of a process that's already running.

```bash
renice -n 10 -p PID             # Change priority of specific PID
renice -n 5 -u username         # Change priority for all user's processes
renice -n 15 -g groupname       # Change priority for all processes in group
```

| Flag | Description |
|------|-------------|
| `-n VALUE` | New nice value |
| `-p PID` | Target specific PID |
| `-u USER` | Target all processes of user |
| `-g GROUP` | Target all processes of group |

### Example Workflow

```bash
# 1. Find a hungry process
ps aux --sort=-%cpu | head -5

# 2. Reduce its priority so it doesn't hog CPU
sudo renice -n 10 -p 72131

# 3. Verify change
ps -o pid,ni,pri,cmd -p 72131
```

---

## Quick Reference

| Task | Command |
|------|---------|
| List all processes | `ps aux` |
| List with threads | `ps -efL` |
| Find process by name | `pgrep name` |
| Gracefully kill by PID | `kill PID` |
| Force kill by PID | `kill -9 PID` |
| Kill by name | `pkill name` |
| Start with low priority | `nice -n 10 cmd` |
| Lower priority of running process | `renice -n 10 -p PID` |
| Pause a process | `kill -STOP PID` |
| Resume a paused process | `kill -CONT PID` |

