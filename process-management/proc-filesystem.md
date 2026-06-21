# `/proc` — The Process Information Pseudo-Filesystem

> A virtual filesystem that exposes kernel and process data as files. Nothing in `/proc` exists on disk.

---

## What Is `/proc`?

`/proc` is a **virtual filesystem** created by the kernel at boot time. It doesn't store data on your hard drive — it generates files on demand by reading internal kernel structures.

Think of it as the kernel's "control panel" exposed as a directory tree.

---

## Top-Level Entries

When you `ls /proc`, you see a mix of numbered directories and named files:

```bash
root@host:/proc# ls
1     244   4905   67681  72131  791       cpuinfo    fb       kcore      loadavg  net        softirqs   tty
1348  247   4976   67682  72535  acpi      crypto     filesystems  key-users  locks    pagetypeinfo  stat       uptime
...
```

| Type | Examples | What They Are |
|------|----------|---------------|
| **Numbered dirs** | `1`, `72131`, `72535` | One directory per running process (named by PID) |
| **System info files** | `cpuinfo`, `meminfo`, `loadavg` | Kernel/system statistics |
| **Config files** | `cmdline`, `filesystems` | Boot and runtime configuration |
| **Kernel internals** | `interrupts`, `iomem`, `kallsyms` | Hardware and kernel debugging data |

### Useful Top-Level Files

```bash
cat /proc/cpuinfo          # CPU model, cores, flags
cat /proc/meminfo          # RAM usage breakdown
cat /proc/loadavg          # 1-min, 5-min, 15-min load averages
cat /proc/uptime           # Seconds since boot
cat /proc/version          # Kernel version
ls /proc/net               # Network statistics and config
```

---

## Process Directories (`/proc/<PID>`)

Every running process gets its own directory named by its PID:

```bash
root@host:/proc# cd 1
root@host:/proc/1# ls
arch_status  cmdline   environ  io      loginuid  mounts   oom_adj    personality  sessionid  stat    timens_offsets
attr         comm      exe      ksm...  map_files mountstats oom_score  projid_map  setgroups  statm   timers
auxv         coredump... fd     latency maps      net      oom_sc...  root        smaps      status  timerslack_ns
cgroup       cpuset    fdinfo   limits  mem       ns       pagemap    sched       smaps_...   syscall uid_map
```

### Key Files Inside a PID Directory

| File | What It Contains |
|------|------------------|
| `cmdline` | Command that started the process, with all arguments (null-separated) |
| `cwd` | Symlink to current working directory |
| `exe` | Symlink to the actual executable file |
| `environ` | Environment variables (null-separated) |
| `fd/` | Directory of symlinks to all open file descriptors |
| `maps` | Memory mappings (libraries, heap, stack regions) |
| `status` | Human-readable process state, memory, signals, capabilities |
| `statm` | Memory statistics in pages (simpler than `status`) |
| `stat` | Raw process statistics for tools like `ps` |
| `task/` | Subdirectory containing one entry per thread |

---

## Example: Inspecting PID 1 (systemd)

```bash
cat /proc/1/cmdline
# Output: /usr/lib/systemd/systemd--system--deserialize=94
```

The kernel replaced null terminators, so arguments run together. This shows systemd was started with `--system` and `--deserialize=94` flags.

```bash
cat /proc/1/statm
# Output: 5573 2848 1920 11 0 848 0
```

| Position | Field | Value | Meaning |
|----------|-------|-------|---------|
| 1 | Total program size | 5573 | Total virtual memory pages |
| 2 | Resident set | 2848 | Pages in physical RAM |
| 3 | Shared pages | 1920 | Shared memory pages |
| 4 | Text (code) | 11 | Executable code pages |
| 5 | Library | 0 | Library pages (deprecated) |
| 6 | Data + Stack | 848 | Data and stack pages |
| 7 | Dirty pages | 0 | Pages waiting to be written |

> Page size is typically 4 KB. So `5573 × 4 = ~22,292 KB` total virtual size.

```bash
cat /proc/1/status
```

Key fields from the output:

| Field | Value | Meaning |
|-------|-------|---------|
| `Name` | `systemd` | Process name |
| `State` | `S (sleeping)` | Currently sleeping, waiting for events |
| `Tgid` | `1` | Thread Group ID = PID of main thread |
| `Pid` | `1` | This process/thread's PID |
| `PPid` | `0` | Parent PID (0 = started by kernel at boot) |
| `Uid` | `0 0 0 0` | Real, effective, saved, filesystem UID (all root) |
| `Gid` | `0 0 0 0` | Same for group IDs |
| `VmSize` | `22292 kB` | Total virtual memory |
| `VmRSS` | `11392 kB` | Physical RAM used |
| `Threads` | `1` | Number of threads in this process |
| `voluntary_ctxt_switches` | `18399` | Times process gave up CPU voluntarily |
| `nonvoluntary_ctxt_switches` | `423` | Times kernel preempted process |

---

## Quick Reference

| Task | File/Path |
|------|-----------|
| What command started a process? | `/proc/<PID>/cmdline` |
| Where is the executable? | `/proc/<PID>/exe` |
| What directory is it in? | `/proc/<PID>/cwd` |
| What files does it have open? | `ls /proc/<PID>/fd/` |
| What are its environment variables? | `/proc/<PID>/environ` |
| Human-readable process info | `/proc/<PID>/status` |
| Memory usage numbers | `/proc/<PID>/statm` |
| Per-thread info | `/proc/<PID>/task/` |
| CPU info | `/proc/cpuinfo` |
| RAM info | `/proc/meminfo` |
| System load | `/proc/loadavg` |
| Kernel version | `/proc/version` |

