# Standard Streams: STDIN, STDOUT, and STDERR

> Understanding how Linux handles input and output streams, and how to redirect them.

---

## The Three Standard Streams

In Linux, every process has three standard communication channels:

| Stream | Number | Name | Default Destination | Description |
|--------|--------|------|---------------------|-------------|
| Standard Input | `0` | **STDIN** | Keyboard | Where programs read input from |
| Standard Output | `1` | **STDOUT** | Terminal Screen | Where programs write normal output |
| Standard Error | `2` | **STDERR** | Terminal Screen | Where programs write error messages |

By default, both STDOUT and STDERR print to your terminal screen.

---

## Redirection Operators

### `>` — Redirect STDOUT to a File (Overwrite)

Redirects the normal output (STDOUT, stream 1) to a file. Overwrites the file if it exists.

```bash
ls -l > output.txt           # Save directory listing to file
ls -l 1> output.txt          # Same thing (1 is STDOUT, optional)
echo "Hello" > hello.txt     # Write text to file
cat file1 file2 > combined   # Combine files
```

> **Without `>`**: output prints to terminal.<br>
> **With `>`**: output goes to the file instead.

---

### `>>` — Redirect STDOUT to a File (Append)

Same as `>`, but appends to the file instead of overwriting.

```bash
echo "Line 1" > log.txt      # Create file with "Line 1"
echo "Line 2" >> log.txt     # Append "Line 2" to file
echo "Line 3" >> log.txt     # Append "Line 3" to file
```

---

### `2>` — Redirect STDERR to a File

Redirects only error messages (stream 2) to a file.

```bash
ls /nonexistent 2> errors.txt     # Redirect error to file
ls /exists /nonexistent 2> err.txt  # Error saved, normal output still prints
```

**Example showing the difference:**

```bash
# Both stdout and stderr go to terminal
ls /root
# Output: ls: cannot open directory '/root': Permission denied

# Redirect only errors to a file
ls /root 2> errors.txt
# Output: (nothing on screen)
# File errors.txt now contains: ls: cannot open directory '/root': Permission denied

# Redirect only stdout to a file (errors still print)
ls /root 1> output.txt
# Output: ls: cannot open directory '/root': Permission denied
```

---

### `&>` — Redirect Both STDOUT and STDERR

Redirects both normal output and errors to the same file.

```bash
command &> output.txt         # Both stdout and stderr go to file
command > output.txt 2>&1     # Same thing (older syntax)
```

| Syntax | Meaning |
|--------|---------|
| `> file` | Redirect STDOUT to file |
| `1> file` | Same as above (explicit) |
| `2> file` | Redirect STDERR to file |
| `&> file` | Redirect both STDOUT and STDERR to file |
| `> file 2>&1` | Redirect both (alternative syntax) |

---

### Separating STDOUT and STDERR

Send normal output and errors to different files:

```bash
ls /root /home > output.txt 2> errors.txt
# Normal output → output.txt
# Errors → errors.txt
```

---

### `/dev/null` — The Black Hole

Discard unwanted output by redirecting to `/dev/null`:

```bash
command > /dev/null           # Discard normal output
command 2> /dev/null          # Discard errors only
command &> /dev/null          # Discard everything
```

---

### `<` — Redirect STDIN from a File

Feed a file as input to a command.

```bash
cat < input.txt               # Read input.txt and display it
sort < unsorted.txt           # Sort contents of file
wc -l < file.txt              # Count lines in file
```

| Syntax | Meaning |
|--------|---------|
| `< file` | Take input from file instead of keyboard |
| `0< file` | Same as above (0 is STDIN, optional) |

---

## The Pipe Operator (`|`)

The pipe sends the **STDOUT** of one command as **STDIN** to another command.

```bash
command1 | command2           # Pipe stdout of cmd1 to stdin of cmd2
```

### How It Works

```bash
ls -l | grep ".txt"           # List files, then filter for .txt files
cat file.txt | sort           # Display file contents, then sort lines
ps aux | grep firefox         # List processes, find firefox
```

**Important**: Only STDOUT flows through the pipe. STDERR does **not**.

```bash
# Example: stderr does NOT go through the pipe
ls /root /home | grep "something"
# The error about /root still prints to terminal
# Only the successful output from /home goes through the pipe
```

To include errors in the pipe:

```bash
ls /root /home 2>&1 | grep "Permission"
# Now both stdout and stderr flow through the pipe
```

---

## Combining Redirection and Pipes

```bash
# Pipe stdout, save errors to file
ls /root /home 2> errors.txt | sort

# Pipe both stdout and stderr
cat file.txt 2>&1 | tee log.txt

# Chain multiple pipes
cat file.txt | grep "error" | sort | uniq | wc -l
# 1. Read file
# 2. Filter lines containing "error"
# 3. Sort them
# 4. Remove duplicates
# 5. Count unique lines
```

---

## Logical AND (`&&`)

Run the second command **only if** the first command succeeds (exits with status `0`).

```bash
command1 && command2            # Run cmd2 only if cmd1 succeeds
```

A command "succeeds" when it exits with status code `0`. It "fails" with a non-zero exit code. Importantly, `&&` checks the **exit status**, not whether anything was printed to STDERR.

### Examples

```bash
# Both succeed — both run
mkdir newdir && cd newdir
# Creates directory, then enters it

# First fails — second does NOT run
ls /nonexistent && echo "Success"
# ls exits with error → echo never runs
# Output: ls: cannot access '/nonexistent': No such file or directory

# Redirecting stderr does NOT make a failing command succeed
ls /nonexistent 2> /dev/null && echo "Still runs?"
# Output: (nothing — stderr is hidden, but ls still failed)
# echo does NOT run because ls returned non-zero exit status
```

### Key Point: Exit Status vs. STDERR

| What Happens | Exit Status | `&&` Behavior |
|--------------|-------------|---------------|
| Command prints to STDERR | Could be `0` or non-zero | Depends on exit code |
| Command has non-zero exit | Non-zero | Next command **skipped** |
| Command has zero exit | `0` | Next command **runs** |

```bash
# Example: grep returns 1 when no match found
grep "xyz" file.txt && echo "Found"
# If "xyz" not in file: grep exits with 1 → echo does NOT run

# Even if you hide the error message
grep "xyz" file.txt 2> /dev/null && echo "Found"
# Still: grep exits with 1 → echo does NOT run
```

> **Rule**: `&&` cares about **exit status** (`$?`), not about what was printed to STDERR or STDOUT.

---

## Quick Reference

| Operator | Description | Example |
|----------|-------------|---------|
| `>` | Redirect STDOUT (overwrite) | `ls > file.txt` |
| `>>` | Redirect STDOUT (append) | `echo "hi" >> file.txt` |
| `1>` | Redirect STDOUT (explicit) | `ls 1> file.txt` |
| `2>` | Redirect STDERR | `cmd 2> errors.txt` |
| `&>` | Redirect both STDOUT and STDERR | `cmd &> all.txt` |
| `> file 2>&1` | Redirect both (alt syntax) | `cmd > file 2>&1` |
| `<` | Redirect STDIN from file | `sort < file.txt` |
| `\|` | Pipe STDOUT to next command | `ls \| grep txt` |
| `&&` | Run next command only if first succeeds | `mkdir dir && cd dir` |

---

## File Descriptors Cheat Sheet

| Number | Stream | Default | Redirect With |
|--------|--------|---------|---------------|
| `0` | STDIN | Keyboard | `<` or `0<` |
| `1` | STDOUT | Terminal | `>` or `1>` |
| `2` | STDERR | Terminal | `2>` |

> **Tip**: Remember: **0** = In, **1** = Out, **2** = Error
