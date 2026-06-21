# Interactive Process Viewers: `top` and `htop`

> Real-time, interactive tools for monitoring processes and system resources.

---

## `top` — Built-In Process Viewer

`top` is installed by default on virtually every Linux system. It shows a continuously updating list of processes.

```bash
top
```

### Key Bindings Inside `top`

| Key | Action |
|-----|--------|
| `q` | Quit |
| `h` or `?` | Help |
| `k` | Kill a process (prompts for PID) |
| `r` | Renice a process (prompts for PID and value) |
| `Space` | Immediately refresh display |
| `M` | Sort by memory usage |
| `P` | Sort by CPU usage |
| `T` | Sort by cumulative time |
| `c` | Toggle command line display |
| `1` | Show individual CPU cores |

### Columns in `top`

| Column | Meaning |
|--------|---------|
| `PID` | Process ID |
| `USER` | Owner |
| `PR` | Priority (kernel-calculated) |
| `NI` | Nice value |
| `VIRT` | Virtual memory |
| `RES` | Resident (physical RAM) |
| `SHR` | Shared memory |
| `S` | State (R/S/T/Z) |
| `%CPU` | CPU percentage |
| `%MEM` | Memory percentage |
| `TIME+` | Cumulative CPU time |
| `COMMAND` | Command name |

---

## `htop` — Enhanced Interactive Viewer

`htop` is a more user-friendly, colorful alternative to `top`. It requires installation.

```bash
sudo apt install htop   # Debian/Ubuntu
sudo dnf install htop   # Fedora
```

Launch:
```bash
htop
```

### `htop` Columns Explained

| Column | Meaning |
|--------|---------|
| **PID** | Process ID. Can also represent **Thread ID** when showing threads — remember from `ps -efL`, a single process has one PID but multiple LWPs. `htop` can display these as if they were separate PIDs. Press `H` to toggle thread visibility. |
| **USER** | Account owner running the process |
| **PRI** | **Priority** — the internal kernel priority rating. Lower numbers mean higher execution priority. The kernel calculates this dynamically based on the process's behavior and nice value |
| **NI** | **Nice value** — a manual priority offset you can control. Ranges from `-20` (highest priority) to `19` (lowest priority). `0` is the standard default. You change this with `renice` or F7/F8 in `htop` |
| **VIRT** | **Virtual memory** — total address space the process reserves (including shared libraries, mapped files, and swapped data). This can be **massively larger** than actual RAM use. Languages like Go, Java, and Electron apps routinely reserve gigabytes of virtual space |
| **RES** | **Resident memory** — the actual physical RAM the process is using **right now**. This is the real memory consumption |
| **SHR** | **Shared memory** — portion of RES that could be shared with other processes (common libraries, system frameworks) |
| **S** | **State** — process state: `R` (Running), `S` (Sleeping), `T` (Stopped), `Z` (Zombie) |
| **CPU%** | Current CPU usage percentage |
| **MEM%** | Current physical RAM usage percentage. Calculated as: `(RES / Total Physical RAM) × 100`. If opencode shows `RES=752M` and `MEM%=9.7%`, your total RAM is approximately `752 ÷ 0.097 ≈ 7,752 MB (~8 GB)` |
| **TIME+** | Total processor execution time since the process started |
| **Command** | The command that launched the process |

---

## PRI vs NI — The Difference

| | **PRI (Priority)** | **NI (Nice)** |
|--|-------------------|---------------|
| **Controlled by** | Linux kernel | You (the user) |
| **Range** | Dynamic, kernel-defined | `-20` to `+19` |
| **What it does** | Real-time scheduling decision | Baseline hint to the kernel |
| **Can you change it?** | No — kernel adjusts automatically | Yes — via `nice`, `renice`, or F7/F8 in `htop` |

**How they work together:**

```
[ Your NI Value ] ---> ( Kernel Math + Behavior Analysis ) ---> [ Final PRI ]
```

- You set `NI` as a request
- The kernel uses that as a baseline, then adjusts `PRI` dynamically based on whether the process is idle, I/O bound, or CPU hungry
- A sleeping process might have its PRI lowered; a compute-heavy process might have it raised

**Golden rule**: You control `NI`. The kernel controls `PRI`.

---

## VIRT vs RES vs SHR — Memory Breakdown

Think of a chef in a kitchen:

```
+-----------------------------------------------------------+
| VIRT: Chef's Recipe Book (massive, theoretical capacity)  |
|   +---------------------------------------------------+   |
|   | RES: Ingredients on the counter (physical, real)  |   |
|   |   +---------------------------------------------+ |   |
|   |   | SHR: Shared spices (other chefs use too)    | |   |
|   |   +---------------------------------------------+ |   |
|   +---------------------------------------------------+   |
+-----------------------------------------------------------+
```

| Metric | Analogy | What It Measures |
|--------|---------|------------------|
| **VIRT** | Recipe book | Everything the process *could* use: code, data, libraries, mapped files, unallocated reservations. Can be **much larger** than physical RAM. Not an indicator of real memory pressure |
| **RES** | Ingredients on counter | Actual physical RAM currently occupied. **This is the real memory cost** |
| **SHR** | Shared spices | Part of RES that is shared with other processes. If opencode uses `RES=752M` and `SHR=101M`, then `752 - 101 = 651M` is exclusive to opencode |
| **MEM%** | Slice of the pie | `RES` expressed as a percentage of total system RAM. Tells you: "How much of my hardware is this eating?" |

> **Don't panic over high VIRT**: A program showing 72.5G VIRT is not using 72.5 GB of RAM. It's just reserved address space. Only RES matters for memory pressure.

---

## `htop` Interactive Controls

| Key | Action |
|-----|--------|
| `↑ ↓` or `PgUp/PgDn` | Navigate process list |
| `Space` | Tag/select a process |
| `F1` or `h` | Help |
| `F2` or `S` | Setup (colors, columns, meters) |
| `F3` or `/` | Search for a process |
| `F4` or `\` | Filter processes |
| `F5` or `t` | Tree view (parent-child relationships) |
| `F6` | Sort by column |
| `F7` or `]` | Decrease nice value (raise priority — needs root) |
| `F8` or `[` | Increase nice value (lower priority) |
| `F9` or `k` | Kill selected process |
| `F10` or `q` | Quit |
| `H` | Toggle thread visibility |
| `K` | Toggle kernel thread visibility |
| `I` | Invert sort order |
| `u` | Filter by user |

---

## `top` vs `htop`

| Feature | `top` | `htop` |
|---------|-------|--------|
| **Availability** | Pre-installed everywhere | Needs installation |
| **Interface** | Text-only, less intuitive | Colorful, mouse-friendly |
| **Tree view** | Available (`V`) | Built-in (`F5`) |
| **Kill/Renice** | Keystrokes (`k`, `r`) | Function keys (`F9`, `F7/F8`) |
| **Search/Filter** | Limited | Full search and filter |
| **Customization** | Runtime flags | Interactive setup menu |
| **CPU/Memory bars** | Numbers only | Visual bar graphs |
| **Select multiple** | No | Yes (`Space` to tag) |

> **Recommendation**: Learn `top` because it's always available in emergencies. Use `htop` for daily work because it's faster and friendlier.

---

## Quick Reference

| Task | Command / Key |
|------|---------------|
| Launch basic viewer | `top` |
| Launch enhanced viewer | `htop` |
| Sort by CPU | `P` (top), `F6` then CPU (htop) |
| Sort by memory | `M` (top), `F6` then MEM (htop) |
| Kill a process | `k` then PID (top), `F9` (htop) |
| Change nice value | `r` then PID then value (top), `F7`/`F8` (htop) |
| Show tree view | `V` (top), `F5` (htop) |
| Show/hide threads | — (top), `H` (htop) |
| Show per-CPU stats | `1` (top), enabled in setup (htop) |

