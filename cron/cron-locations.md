# Cron Locations and Structure

> Where cron jobs live on Linux, how the system organizes them, and the differences between system-wide and user-specific schedules.

---

## Where Cron Reads Jobs From

Cron collects jobs from multiple locations:

```text
/etc/crontab              ← System master schedule
/etc/cron.d/*             ← Package-specific schedules
/var/spool/cron/crontabs/* ← User-specific schedules
```

Indirectly, through `run-parts`, it also executes scripts from:

```text
/etc/cron.hourly/
/etc/cron.daily/
/etc/cron.weekly/
/etc/cron.monthly/
/etc/cron.yearly/
```

---

## `/etc/crontab` — The System Master Schedule

The main system-wide crontab. Managed by the administrator.

```bash
cat /etc/crontab
```

Example from a typical Ubuntu system:

```cron
17 * * * * root run-parts --report /etc/cron.hourly
25 6 * * * root run-parts --report /etc/cron.daily
47 6 * * 7 root run-parts --report /etc/cron.weekly
52 6 1 * * root run-parts --report /etc/cron.monthly
```

### System Crontab Format

Unlike user crontabs, the system crontab has **6 fields** — it includes a **username**:

```text
Minute Hour Day Month Weekday User Command
```

```cron
0 2 * * * root backup.sh
```

The command `backup.sh` runs as `root`, regardless of who owns the file.

---

## `/etc/cron.d/` — Package Schedules

Individual files for specific packages or applications. Cron reads every file in this directory.

```bash
ls /etc/cron.d/
```

```text
docker
certbot
anacron
logrotate
```

Example file `/etc/cron.d/certbot`:

```cron
0 */12 * * * root certbot renew
```

> **Why it exists**: Packages install their own cron files here instead of modifying `/etc/crontab`. This keeps the system organized — removing a package removes its cron job automatically.

### `/etc/crontab` vs `/etc/cron.d`

| | `/etc/crontab` | `/etc/cron.d/` |
|--|----------------|-----------------|
| **Purpose** | Central master schedule | One file per package/app |
| **Managed by** | Administrator | Packages (apt, dpkg, etc.) |
| **Format** | 6 fields (includes username) | 6 fields (includes username) |
| **Analogy** | Main switchboard | Individual circuit breakers |

---

## `/var/spool/cron/crontabs/` — User Crontabs

Personal schedules for individual users. Created and edited through the `crontab` command.

```bash
ls /var/spool/cron/crontabs/
```

```text
aniruddha_ghosh
```

### Managing User Crontabs

```bash
crontab -e          # Edit your crontab (opens in default editor)
crontab -l          # List your current crontab
crontab -r          # Remove your crontab entirely
crontab file.txt    # Install a crontab from a file
```

### User Crontab Format

**5 fields** — no username needed (cron already knows who owns the file):

```text
Minute Hour Day Month Weekday Command
```

```cron
*/5 * * * * echo hello >> /tmp/test.log
```

> The command runs as the user who owns the crontab.

### System vs User Crontabs

| Aspect | System (`/etc/crontab`, `/etc/cron.d/`) | User (`/var/spool/cron/crontabs/`) |
|--------|----------------------------------------|-------------------------------------|
| **Fields** | 6 (includes username) | 5 (no username) |
| **Runs as** | Whatever user is specified | The crontab owner |
| **Edited via** | Direct file edit (`nano`, `vim`) | `crontab -e` |
| **Scope** | System-wide | User-only |
| **Permissions** | Root-managed | User-managed |

---

## The Preset Directories: `hourly`, `daily`, `weekly`, `monthly`

Special directories containing scripts that run at predefined intervals. Triggered by entries in `/etc/crontab` using `run-parts`.

### `/etc/cron.hourly/`

Scripts executed **hourly** at minute 17:

```cron
17 * * * * root run-parts --report /etc/cron.hourly
```

### `/etc/cron.daily/`

Scripts executed **daily** at 06:25:

```cron
25 6 * * * root run-parts --report /etc/cron.daily
```

### `/etc/cron.weekly/`

Scripts executed **weekly** on Sunday at 06:47:

```cron
47 6 * * 7 root run-parts --report /etc/cron.weekly
```

### `/etc/cron.monthly/`

Scripts executed **monthly** on the 1st at 06:52:

```cron
52 6 1 * * root run-parts --report /etc/cron.monthly
```

### Flow Diagram

```text
cron daemon
    │
    ├── /etc/crontab
    │      │
    │      ├── 17 * * * *  → run-parts --report /etc/cron.hourly
    │      │                                     │
    │      │                                     ├── script-a
    │      │                                     ├── script-b
    │      │                                     └── script-c
    │      │
    │      ├── 25 6 * * *  → run-parts --report /etc/cron.daily
    │      │                                     │
    │      │                                     ├── apt
    │      │                                     ├── logrotate
    │      │                                     └── man-db
    │      │
    │      ├── 47 6 * * 7  → run-parts --report /etc/cron.weekly
    │      └── 52 6 1 * *  → run-parts --report /etc/cron.monthly
    │
    ├── /etc/cron.d/
    │      ├── docker
    │      ├── certbot
    │      └── ...
    │
    └── /var/spool/cron/crontabs/
           ├── aniruddha_ghosh
           └── ...
```

---

## `run-parts` — Running All Scripts in a Directory

A utility that executes every executable file inside a directory, one after another.

```bash
run-parts /etc/cron.daily
```

If `/etc/cron.daily/` contains:

```text
apt
logrotate
man-db
```

Then `run-parts` effectively runs:

```bash
/etc/cron.daily/apt
/etc/cron.daily/logrotate
/etc/cron.daily/man-db
```

### Why `run-parts` Exists

Without it, `/etc/crontab` would become unwieldy:

```cron
# Bad: one entry per script
25 6 * * * root /etc/cron.daily/apt
25 6 * * * root /etc/cron.daily/logrotate
25 6 * * * root /etc/cron.daily/man-db
```

With `run-parts`, a single entry handles all daily scripts:

```cron
# Good: one entry for all
25 6 * * * root run-parts --report /etc/cron.daily
```

### `--report` Flag

```bash
run-parts --report /etc/cron.daily
```

Without `--report`, scripts run silently. With it, `run-parts` prepends output with the script name:

```text
/etc/cron.daily/backup:
Backup completed
```

This makes troubleshooting much easier — you know which script produced which output.

---

## `cd / &&` in System Crontabs

Ubuntu's crontab often uses:

```bash
cd / && run-parts --report /etc/cron.daily
```

**Why?**

Cron jobs may start from an unpredictable working directory. By explicitly running:

```bash
cd /
```

first, you guarantee the scripts run from a known, consistent location. Scripts inside `/etc/cron.daily/` can safely use relative paths.

---

## Complete Mental Model

```text
Cron is NOT a scheduling language.
It is a 5-column time filter that asks:

  "Does current minute match?"
  "Does current hour match?"
  "Does current day match?"
  "Does current month match?"
  "Does current weekday match?"

If all required fields match → execute command.
```

---

## Quick Reference

| Location | Purpose | Format |
|----------|---------|--------|
| `/etc/crontab` | System master schedule | 6 fields (includes user) |
| `/etc/cron.d/*` | Package-specific schedules | 6 fields (includes user) |
| `/var/spool/cron/crontabs/<user>` | User-specific schedules | 5 fields (no user) |
| `/etc/cron.hourly/` | Scripts run every hour | Executable scripts |
| `/etc/cron.daily/` | Scripts run daily | Executable scripts |
| `/etc/cron.weekly/` | Scripts run weekly | Executable scripts |
| `/etc/cron.monthly/` | Scripts run monthly | Executable scripts |

| Command | Purpose |
|---------|---------|
| `crontab -e` | Edit your user crontab |
| `crontab -l` | List your user crontab |
| `crontab -r` | Remove your user crontab |
| `sudo crontab -e -u username` | Edit another user's crontab |

