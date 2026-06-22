# Cron Syntax and Expressions

> How to read and write cron schedules — the 5-field format, wildcards, intervals, and common gotchas.

---

## What Is Cron?

Cron is a Linux daemon (background service) that executes scheduled tasks automatically. Every minute, it wakes up, checks all configured schedules, and runs any command whose time conditions match the current moment.

```python
while True:
    current_time = get_time()
    for job in all_cron_jobs:
        if matches(job.schedule, current_time):
            run(job.command)
    sleep(60)
```

---

## The 5-Field Format

A standard cron expression has exactly 5 fields:

```text
* * * * *
│ │ │ │ │
│ │ │ │ └── Day of Week  (0-7, Sunday is 0 or 7)
│ │ │ └──── Month        (1-12)
│ │ └────── Day of Month (1-31)
│ └──────── Hour         (0-23)
└────────── Minute       (0-59)
```

```text
Minute Hour DayOfMonth Month DayOfWeek
```

---

## The Asterisk `*`

`*` means **any valid value** — the field is not restricted.

```cron
* * * * *
```

Translation:

```text
Every minute
Every hour
Every day of month
Every month
Every weekday
```

Result: **Runs every minute**.

---

## Basic Examples

### Every Hour on the Hour

```cron
0 * * * *
```

Runs at `01:00`, `02:00`, `03:00` ...

---

### Every Half Hour

```cron
30 * * * *
```

Runs at `01:30`, `02:30`, `03:30` ...

---

### Daily at a Specific Time

```cron
0 5 * * *
```

Runs at **5:00 AM every day**.

---

### Weekly on a Specific Day

```cron
0 5 * * 1
```

Runs at **5:00 AM every Monday**.

---

## Weekday Values

| Value | Day |
|-------|-----|
| `0` | Sunday |
| `1` | Monday |
| `2` | Tuesday |
| `3` | Wednesday |
| `4` | Thursday |
| `5` | Friday |
| `6` | Saturday |
| `7` | Sunday |

Both `0` and `7` represent Sunday.

---

## Intervals with `*/n`

Run every N units within a field.

### Every 5 Minutes

```cron
*/5 * * * *
```

Runs at minute `0`, `5`, `10`, `15` ... `55`.

---

### Every 10 Minutes

```cron
*/10 * * * *
```

Runs at `00`, `10`, `20`, `30`, `40`, `50`.

---

### Every 2 Hours

```cron
0 */2 * * *
```

Runs at `00:00`, `02:00`, `04:00`, `06:00` ...

---

### Every 3 Hours on the 15th Minute

```cron
15 */3 * * *
```

Runs at `00:15`, `03:15`, `06:15` ...

---

## Multiple Values

Use commas to list specific values.

### Twice Daily

```cron
0 9,18 * * *
```

Runs at `09:00` and `18:00`.

---

### Every 15 Minutes During Business Hours

```cron
*/15 9-17 * * 1-5
```

Runs every 15 minutes, from 9 AM to 5 PM, Monday through Friday.

---

## Ranges

Use hyphens for continuous ranges.

### Every Hour During the Workday

```cron
0 9-17 * * *
```

Runs at `09:00`, `10:00`, `11:00` ... `17:00`.

---

### Weekdays Only

```cron
0 9 * * 1-5
```

Runs at 9:00 AM, Monday through Friday.

---

## Complex Expressions

```cron
*/2 */3 1-10 2-8/2 7
```

| Field | Value | Meaning |
|-------|-------|---------|
| Minute | `*/2` | Every 2 minutes (`0,2,4...58`) |
| Hour | `*/3` | Every 3rd hour (`0,3,6...21`) |
| Day of Month | `1-10` | Days 1 through 10 |
| Month | `2-8/2` | Every 2nd month from Feb to Aug (`2,4,6,8`) |
| Day of Week | `7` | Sunday |

Translation:

> Every 2 minutes, during every 3rd hour, on days 1-10, during February/April/June/August, on Sundays.

---

## The Gotcha: Day of Month + Day of Week

Many cron implementations use **OR logic** between the Day of Month and Day of Week fields.

```cron
0 2 1 * 1
```

You might expect: "Run at 2 AM on the 1st of the month **AND** on Mondays."

But cron actually evaluates it as: "Run at 2 AM on the 1st of the month **OR** on Mondays."

So this job runs on **both** the 1st of every month **and** every Monday.

> **Rule**: If both Day of Month and Day of Week are specified (not `*`), cron treats them as an OR condition, not AND. To get AND behavior, you typically need to handle it inside the script itself.

---

## Mental Model

Cron is not a scheduling language. It is a **5-column time filter**.

It does not think:

```text
"Run every 5 minutes"
```

It thinks:

```text
"Does current minute match?
Does current hour match?
Does current day match?
Does current month match?
Does current weekday match?"
```

If all conditions are satisfied → execute the command.

This mental model makes debugging complex expressions much easier.

---

## Quick Reference

| Expression | Meaning |
|------------|---------|
| `* * * * *` | Every minute |
| `0 * * * *` | Every hour on the hour |
| `*/5 * * * *` | Every 5 minutes |
| `0 0 * * *` | Daily at midnight |
| `0 5 * * *` | Daily at 5 AM |
| `0 9 * * 1-5` | Weekdays at 9 AM |
| `0 0 * * 0` | Weekly on Sunday at midnight |
| `0 0 1 * *` | Monthly on the 1st at midnight |
| `0 0 1 1 *` | Yearly on January 1st at midnight |
| `*/10 9-17 * * 1-5` | Every 10 minutes, 9 AM–5 PM, weekdays |

