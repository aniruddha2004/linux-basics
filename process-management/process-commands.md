# Process Management Commands

> Practical commands for inspecting, controlling, and adjusting running processes.

---

## `ps` — Process Status

Display information about active processes. `ps` has multiple syntax styles — Unix (`-`), BSD (no dash), and GNU (`--`).

### Common Flags

| Flag | Style | Meaning |
|------|-------|---------|
| `a` | BSD | Show processes for all users |
| `u` | BSD | User-oriented format (shows CPU%, MEM%, etc.) |
| `x` | BSD | Show processes not attached to a terminal (daemons, background) |
| `e` | Unix | Show environment variables after the command |
| `f` | Unix | Full format listing with ASCII art tree |
| `L` | Both | Show threads (LWPs) — displays one line per thread |

### Common Combinations

```bash
ps aux                           # All processes for all users, detailed
ps -ef                           # All processes, full format listing
ps aux | grep firefox           # Find a specific process
ps -fL -C firefox               # Show threads of a specific command
ps -efL | grep opencode         # Show all threads of a process (from theory)
```

### `ps aux` Output Columns

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

