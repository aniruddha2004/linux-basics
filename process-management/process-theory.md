# Processes and Threads — Theory

> Core concepts behind process management in Linux: what processes are, how they live and die, and how threads fit into the picture.

---

## What Is a Process?

A **process** is a running instance of a program. When you double-click an application or run a command, the operating system loads the program from disk into memory and starts executing it — that executing instance is a process.

Think of it this way:
- **Program** = recipe (static file on disk)
- **Process** = actually cooking the recipe (running in memory)

Every process is identified by a unique number called a **PID** (Process ID).

---

## Components of a Process

When a process is created, the kernel allocates several data structures to track and manage it:

| Component | Description |
|-----------|-------------|
| **PID** | Unique Process Identifier assigned by the kernel |
| **PPID** | Parent Process ID — the process that spawned this one |
| **Memory Space** | Virtual address space containing code, data, heap, stack |
| **File Descriptors** | Table of open files (stdin=0, stdout=1, stderr=2) |
| **CPU Registers** | Current values of CPU registers (program counter, stack pointer, etc.) |
| **Environment Variables** | Variables like `PATH`, `HOME`, `USER` |
| **Signal Handlers** | Functions that respond to signals (SIGTERM, SIGKILL, etc.) |
| **Scheduling Info** | Priority, nice value, CPU time used |
| **Thread(s)** | At least one thread of execution (the main thread) |

---

## Process Lifecycle

A process goes through distinct states during its lifetime:

```
       ┌─────────────┐
       │   Created   │  ← fork() / exec()
       └──────┬──────┘
              │
              ▼
       ┌─────────────┐
       │    Ready    │  ← Waiting for CPU
       └──────┬──────┘
              │ Scheduler picks it
              ▼
       ┌─────────────┐
       │   Running   │  ← Executing on CPU
       └──────┬──────┘
              │
    ┌─────────┼─────────┐
    │         │         │
    ▼         ▼         ▼
Blocked   Exit/     Preempted
(waiting) Terminated   (back to Ready)
(I/O, etc)
```

### Process States

| State | Meaning |
|-------|---------|
| **Running** | Currently executing on the CPU |
| **Ready** | Waiting in queue for CPU time |
| **Blocked / Waiting** | Paused, waiting for I/O or an event (disk read, network, input) |
| **Terminated / Zombie** | Finished execution, waiting for parent to read exit status |
| **Stopped** | Suspended (e.g., by Ctrl+Z or SIGSTOP signal) |

### Zombie Processes

When a child process finishes, its PID stays in the process table until the parent calls `wait()` to read its exit status. During this brief period, the child is a **zombie**. It consumes almost no resources but holds its PID.

If the parent dies without reading the status, `init` (PID 1) adopts the zombie and cleans it up. But if a parent ignores its children indefinitely, zombies can accumulate.

---

## Threads

### What Is a Thread?

A **thread** is a single sequence of execution within a process. Every process starts with at least one thread — the **main thread**.

A process can have multiple threads, all sharing the same:
- Memory space (heap, code, global variables)
- Open files and file descriptors
- Environment variables
- Signal handlers

But each thread has its own:
- Program counter (what instruction it's on)
- Register set
- Stack (local variables, function calls)
- Thread ID

### Process vs Thread

| Aspect | Process | Thread |
|--------|---------|--------|
| **Memory** | Own private address space | Shares memory with other threads in the same process |
| **Creation** | Expensive (fork, new memory mapping) | Cheap (just a new stack and registers) |
| **Communication** | Needs IPC (pipes, sockets, shared memory) | Direct — read/write same variables |
| **Crash Impact** | Crashes only itself | Can crash the entire process |
| **Isolation** | Strong — one can't corrupt another | Weak — bugs affect all threads |
| **Context Switch** | Heavy (MMU, caches) | Light (registers + stack only) |
| **OS Scheduling** | Kernel schedules processes | Kernel schedules threads |

> **Simple Analogy**: A process is like a factory building. A thread is like a worker inside that factory. Multiple workers (threads) can work inside the same building (process) and share tools and materials (memory, files). Opening a new building (process) is much more work than hiring a new worker (thread).

---

## LWPs — Lightweight Processes

In Linux, threads are implemented as **Lightweight Processes** (LWPs). The kernel doesn't distinguish much between a single-threaded process and a thread — both are schedulable entities with their own PID-like identifier.

Historically:
- Older systems had heavy processes with threads managed in user space
- Linux made threads first-class citizens at the kernel level using the `clone()` system call
- Every thread gets its own kernel-visible ID

This is why Linux tools often blur the line between "process" and "thread" — at the kernel scheduler level, they are treated similarly.

---

## PIDs, TIDs, and Thread IDs in Linux

### The Confusion

In Linux, there are multiple IDs involved:

| ID | Meaning | Example |
|----|---------|---------|
| **PID** | Process ID — the thread group leader | Main identifier of the process |
| **TID** | Thread ID — unique identifier for each thread | What the kernel uses internally |
| **LWP** | LightWeight Process ID — often same as TID | Visible in tools like `ps -L` |
| **TGID** | Thread Group ID — same as PID of the main thread | Groups all threads of a process |

> **Important**: In single-threaded programs, PID == TID. In multi-threaded programs, each thread has a unique TID but shares the same TGID (process-level PID).

### Why htop Shows PID for Threads

By default, `htop` shows threads as if they were separate processes with their own "PID". This is because Linux exposes threads through the `/proc` filesystem as pseudo-directories under `/proc/<pid>/task/<tid>/`.

Tools like `htop` can optionally hide threads (press `H` or go to Setup) so you only see the main process entries.

**In `htop`:**
- Tree view (`F5`) shows parent-child relationships
- Threads appear indented under their parent process
- The "PID" column for threads actually shows their TID

---

## Reading `ps -efL`

The `-L` flag tells `ps` to show threads (LWPs) separately.

### Your Example: `ps -efL | grep opencode`

```
UID        PID   PPID   LWP  C NLWP STIME TTY      TIME CMD
anirudd+ 72131  71044 72131  2   28 12:38 pts/2  00:00:12 opencode    ← Main thread
anirudd+ 72131  71044 72132  0   28 12:38 pts/2  00:00:01 opencode    ← Thread 2
anirudd+ 72131  71044 72133  0   28 12:38 pts/2  00:00:02 opencode    ← Thread 3
anirudd+ 72131  71044 72134  0   28 12:38 pts/2  00:00:02 opencode    ← Thread 4
...
```

### Column Breakdown

| Column | Meaning | In Your Example |
|--------|---------|-----------------|
| `UID` | User who owns the process | `anirudd+` (aniruddha_ghosh) |
| `PID` | Process ID (actually TGID) | `72131` — same for all rows |
| `PPID` | Parent Process ID | `71044` — the parent that spawned opencode |
| `LWP` | LightWeight Process / Thread ID | `72131`, `72132`, `72133`... — unique per row |
| `C` | CPU utilization percentage | `2` for main thread, `0` for most others |
| `NLWP` | Number of LWPs (threads) in this process | `28` — opencode has 28 threads total |
| `STIME` | Start time | `12:38` |
| `TTY` | Terminal | `pts/2` |
| `TIME` | Cumulative CPU time | Main thread used 12 seconds, others 0-2 |
| `CMD` | Command name | `opencode` |

### What This Tells Us

1. **One process, many threads**: All rows share PID `72131` — that's the process. But each row has a unique LWP — those are the individual threads.

2. **Main thread**: The first row has `PID == LWP` (`72131 == 72131`). This is the thread group leader.

3. **Worker threads**: Rows where `LWP != PID` are worker threads doing concurrent work inside the same process.

4. **CPU distribution**: The main thread consumed the most CPU (`00:00:12`), while most worker threads used minimal time.

5. **Thread count**: `NLWP = 28` tells us opencode is multi-threaded with 28 concurrent execution paths.

---

## Single-Threaded vs Multi-Threaded in `ps`

### Single-Threaded Process

```bash
ps -efL | grep bash

UID        PID   PPID   LWP  C NLWP STIME TTY      TIME CMD
anirudd+  1234   1000  1234  0    1 09:00 pts/0  00:00:00 bash
```

Notice: `PID == LWP` and `NLWP = 1` — only one thread.

### Multi-Threaded Process

```bash
ps -efL | grep firefox

UID        PID   PPID   LWP  C NLWP STIME TTY      TIME CMD
anirudd+  5678   1000  5678  2   15 10:15 ?      00:01:23 firefox
anirudd+  5678   1000  5679  0   15 10:15 ?      00:00:05 firefox
anirudd+  5678   1000  5680  0   15 10:15 ?      00:00:03 firefox
...
```

Notice: Same PID, different LWPs, `NLWP = 15`.

---

## Why Threads Matter

| Benefit | Explanation |
|---------|-------------|
| **Responsiveness** | One thread handles UI while another does heavy computation |
| **Resource Sharing** | Threads communicate easily through shared memory |
| **Economy** | Creating threads is cheaper than creating processes |
| **Scalability** | Multiple threads can run on multiple CPU cores simultaneously |
| **Throughput** | A web server handles many connections with one thread per connection |

But threads also bring complexity:
- **Race conditions** — two threads modify the same variable simultaneously
- **Deadlocks** — threads wait on each other forever
- **Debugging difficulty** — reproducing thread bugs is hard

---

## Signals

A **signal** is a software interrupt delivered to a process by the operating system or another process. Signals notify a process that an event has occurred and require immediate attention.

Think of signals like doorbells — they interrupt whatever the process is doing and demand a response.

### How Signals Work

1. A signal is **sent** to a process (by kernel, another process, or the user)
2. The kernel **delivers** the signal to the target process
3. The process **handles** the signal by:
   - Executing a custom handler (if registered)
   - Performing the default action (terminate, ignore, stop, continue)
   - Ignoring it (for certain signals)

### Common Signals

| Signal | Number | Default Action | Description |
|--------|--------|----------------|-------------|
| `SIGINT` | `2` | Terminate | **Interrupt** — sent by pressing `Ctrl+C`. Polite request to stop. |
| `SIGTERM` | `15` | Terminate | **Terminate** — graceful shutdown request. Process can cleanup before exiting. |
| `SIGKILL` | `9` | Terminate | **Kill** — forceful termination. Cannot be caught, blocked, or ignored. |
| `SIGHUP` | `1` | Terminate | **Hang Up** — sent when terminal disconnects. Often used to reload configs. |
| `SIGSTOP` | `19` | Stop | **Stop** — pauses process execution. Cannot be caught or ignored. |
| `SIGCONT` | `18` | Continue | **Continue** — resumes a stopped process. |
| `SIGTSTP` | `20` | Stop | **Terminal Stop** — sent by pressing `Ctrl+Z`. Pauses process. |
| `SIGQUIT` | `3` | Terminate (core dump) | **Quit** — sent by `Ctrl+\`. Terminates and creates a core dump for debugging. |
| `SIGSEGV` | `11` | Terminate (core dump) | **Segmentation Fault** — illegal memory access. Usually a bug. |
| `SIGCHLD` | `17` | Ignore | **Child Changed** — sent to parent when a child process stops or terminates. |

---

### SIGTERM vs SIGKILL — The Critical Difference

This is one of the most important distinctions in process management.

#### SIGTERM (Signal 15) — "Please stop"

- **Polite request** to terminate
- The process receives it and can decide what to do
- Process can **register a handler** to catch it and perform cleanup:
  - Save unsaved data
  - Close database connections
  - Write logs
  - Delete temp files
  - Notify other services
- If no handler, the default action is termination
- **Graceful shutdown**

```
Process running normally
       │
       ▼
SIGTERM received
       │
       ├── Process has a handler?
       │   ├── Yes → Run cleanup code → Exit
       │   └── No  → Default: terminate immediately
       │
       └── Process ignores it?
           └── Yes → Keeps running (rare)
```

#### SIGKILL (Signal 9) — "You WILL stop"

- **Forceful termination** — no discussion
- The kernel terminates the process **immediately**
- Cannot be caught, blocked, or ignored — guaranteed to work
- Process gets **no chance to clean up**
- No cleanup code runs
- May leave:
  - Corrupted files
  - Orphaned child processes
  - Locked resources
  - Unsaved data lost
- **Use only when SIGTERM fails**

```
Process running normally
       │
       ▼
SIGKILL received
       │
       ▼
Kernel forcefully terminates process
       │
       ▼
Process killed instantly — no cleanup possible
```

#### When to Use Which

| Scenario | Signal | Reason |
|----------|--------|--------|
| Normal shutdown | SIGTERM | Allows cleanup |
| Graceful restart | SIGHUP | Process can reload config without dying |
| User interruption | SIGINT | Standard Ctrl+C behavior |
| Process frozen / ignoring SIGTERM | SIGKILL | Last resort |
| Debugging a crash | SIGQUIT | Creates core dump for analysis |

> **Rule of thumb**: Always try SIGTERM first. Wait a few seconds. Only use SIGKILL if the process refuses to die.

---

### SIGSTOP and SIGCONT — Pause and Resume

Unlike termination signals, these control process execution without killing it.

#### SIGSTOP (Signal 19) — Freeze

- Pauses the process immediately
- Process is moved to **Stopped** state
- Does not consume CPU — frozen in place
- Cannot be caught or ignored (like SIGKILL)
- Sent by `Ctrl+Z` in terminal

```bash
# In terminal
$ sleep 100
^Z                    # Press Ctrl+Z
[1]+  Stopped                 sleep 100

# Process is now frozen, not dead
```

#### SIGCONT (Signal 18) — Unfreeze

- Resumes a previously stopped process
- Moves process back to **Running** or **Ready** state
- Sent by the `fg` or `bg` command

```bash
$ jobs
[1]+  Stopped                 sleep 100

$ bg %1                    # Resume in background
[1]+ sleep 100 &

$ fg %1                    # Bring to foreground
sleep 100
```

#### Stopped vs Terminated

| Aspect | Stopped (SIGSTOP) | Terminated (SIGTERM/SIGKILL) |
|--------|-------------------|------------------------------|
| Memory | Retained | Freed |
| Open files | Kept open | Closed (or abandoned) |
| CPU usage | Zero | Zero (process gone) |
| Can resume? | Yes (SIGCONT) | No |
| PID reused? | No | Yes (eventually) |

---

### SIGINT — The Ctrl+C Signal

Sent when you press `Ctrl+C` in a terminal.

- **Interrupt** — "Stop what you're doing and exit"
- Most interactive programs handle this gracefully
- Can be caught by the process
- Default action: terminate

```bash
$ cat /dev/zero > /dev/null
^C                      # Press Ctrl+C — SIGINT sent

# Process terminates cleanly
```

Some programs override SIGINT to do something else:
- Python REPL: Catches SIGINT, raises `KeyboardInterrupt` exception
- `ping`: Stops pinging, prints summary, exits
- `ffmpeg`: Stops encoding, finalizes output file, exits

---

### Signal Summary Table

| Signal | Triggered By | Can Catch? | Can Ignore? | Use Case |
|--------|-------------|------------|-------------|----------|
| `SIGINT` | `Ctrl+C` | Yes | Yes | User interruption |
| `SIGTERM` | `kill` command | Yes | Yes | Graceful shutdown |
| `SIGKILL` | `kill -9` | **No** | **No** | Force kill |
| `SIGSTOP` | `Ctrl+Z` | **No** | **No** | Pause process |
| `SIGCONT` | `fg`, `bg` | Yes | Yes | Resume process |
| `SIGHUP` | Terminal disconnect | Yes | Yes | Reload config |
| `SIGTSTP` | `Ctrl+Z` | Yes | Yes | Terminal stop (like SIGSTOP but catchable) |
| `SIGQUIT` | `Ctrl+\` | Yes | Yes | Quit + core dump |

---

### Process State Transitions with Signals

```
         ┌──────────┐
         │ Running  │
         └────┬─────┘
              │
    ┌─────────┼─────────┬─────────┐
    │         │         │         │
    ▼         ▼         ▼         ▼
 SIGSTOP   SIGTERM  SIGKILL   SIGINT
(orfrozen)  (or     (or      (or
            SIGINT    SIGKILL) SIGTERM)
    │         │         │         │
    ▼         ▼         ▼         ▼
 ┌──────┐  ┌─────┐  ┌─────┐  ┌─────┐
 │Stopped│  │Zombie│  │Gone │  │Gone │
 └──┬───┘  └─────┘  └─────┘  └─────┘
    │
 SIGCONT
    │
    ▼
 ┌──────────┐
 │ Running  │
 └──────────┘
```

> **SIGTSTP vs SIGSTOP**: `SIGTSTP` (Ctrl+Z) is technically catchable, but most programs let it behave like `SIGSTOP`. Some programs (like `vim`, `less`) catch it to restore terminal state before stopping.

---

## Summary

| Concept | Key Point |
|---------|-----------|
| **Process** | A running program with its own memory space |
| **Thread** | A sequence of execution within a process |
| **PID** | Identifies the process (shared by all its threads) |
| **TID / LWP** | Identifies an individual thread |
| **TGID** | Thread Group ID = PID of the main thread |
| **Linux Threads** | Implemented as LWPs — kernel treats them similarly to processes |
| `ps -efL` | Shows both PID and LWP columns to distinguish process from thread |
| **Signals** | Software interrupts for controlling processes (SIGTERM, SIGKILL, SIGSTOP, etc.) |
| **SIGTERM vs SIGKILL** | SIGTERM asks nicely and allows cleanup; SIGKILL forces immediate death with no cleanup |
| **SIGSTOP / SIGCONT** | Pause and resume a process without terminating it |
