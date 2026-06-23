# Recording Terminal Sessions with `script`

> Record and replay everything typed and displayed in a terminal session.

---

## What Is `script`?

`script` is a built-in Linux utility that records all terminal activity — commands, output, errors — into a file. It is useful for:
- Documenting what you did during a session
- Creating tutorials or reproducible steps
- Saving command output for later review
- Auditing terminal activity

> Think of it as a screen recorder, but for text-only terminal sessions.

---

## Basic Recording

```bash
script test.log
```

You will see:

```text
Script started, output log file is 'test.log'.
```

Now everything you type and see is saved to `test.log`. When you are done:

```bash
exit
```

Or press `Ctrl-D`. You will see:

```text
Script done.
```

### View the Recording

```bash
cat test.log
```

The log contains timestamps, commands, and their outputs exactly as they appeared.

---

## Recording with Timing for Replay

To replay the session in real time (preserving delays and typing speed), record both an output file and a timing file:

```bash
script test.log --timing=time.log
```

Now the terminal records:
- `test.log` — all text output
- `time.log` — timing information for replay

Stop recording with `exit` or `Ctrl-D`.

---

## Replaying a Session with `scriptreplay`

```bash
scriptreplay -s test.log -t time.log
```

| Flag | Meaning |
|------|---------|
| `-s` | Session file (output log) |
| `-t` | Timing file |

The session plays back on your terminal, including pauses and typing speed.

> **Important**: `scriptreplay` requires the timing file. Without it, you can only view the static log with `cat`.

---

## Quick Reference

| Task | Command |
|------|---------|
| Start recording | `script output.log` |
| Stop recording | `exit` or `Ctrl-D` |
| Record with timing | `script output.log --timing=time.log` |
| Replay with timing | `scriptreplay -s output.log -t time.log` |
| View log | `cat output.log` |

---

## Tips

- Use plain `script` when you just need a transcript.
- Use `script --timing` when you need a realistic replay.
- Logs can grow large quickly when running programs like `htop` or long command outputs.
- Clean up old `.log` and `.timing` files to save disk space.
