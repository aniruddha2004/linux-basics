# Tmux Basics

> Simple notes for getting started with tmux sessions, windows, and panes.

---

## What Is Tmux?

Tmux is a terminal multiplexer. It lets you:
- Run multiple terminal sessions inside one window
- Split windows into panes
- Keep sessions running in the background after you disconnect
- Reattach to sessions later

---

## Sessions

A **session** is a container that holds windows and panes. Sessions keep running even when you detach.

### Create a Session

```bash
tmux                           # Create an unnamed session
tmux new -s demo-session       # Create a named session
```

> **Difference**: `tmux` creates a session with a default number name (like `0`). `tmux new -s demo-session` gives it a readable name, which is easier to reattach to later.

### Detach from a Session

Press inside tmux:

```text
Ctrl-b  d
```

This leaves the session running in the background.

### List Sessions

```bash
tmux list-session
```

Example output:

```text
demo-session: 1 windows (created Tue Jun 23 02:11:07 2026)
```

### Reattach to a Session

```bash
tmux attach -t demo-session
```

> `-t` stands for "target". You can also use `-t 0` for unnamed sessions.

---

## Windows

A **window** is like a tab. Each window has its own shell prompt.

| Key Combo | Action |
|-----------|--------|
| `Ctrl-b c` | Create new window |
| `Ctrl-b ,` | Rename current window |
| `Ctrl-b n` | Next window |
| `Ctrl-b p` | Previous window |
| `Ctrl-b w` | List all windows interactively |
| `Ctrl-b d` | Detach from session |

---

## Panes

A **pane** is a split view inside a single window. You can run multiple commands side by side.

| Key Combo | Action |
|-----------|--------|
| `Ctrl-b %` | Split window vertically (two panes left/right) |
| `Ctrl-b :` then type `split-window` | Split horizontally (two panes top/bottom) |

> After `Ctrl-b :`, tmux opens a command line at the bottom where you type `split-window` and press Enter.

### Moving Between Panes

| Key Combo | Action |
|-----------|--------|
| `Ctrl-b ←` | Move left |
| `Ctrl-b →` | Move right |
| `Ctrl-b ↑` | Move up |
| `Ctrl-b ↓` | Move down |

> Use the arrow keys after `Ctrl-b` to jump between panes.

---

## Quick Reference

| Task | Command / Key |
|------|---------------|
| Start tmux | `tmux` |
| Start named session | `tmux new -s <name>` |
| Detach | `Ctrl-b d` |
| List sessions | `tmux list-session` |
| Reattach | `tmux attach -t <name>` |
| New window | `Ctrl-b c` |
| Rename window | `Ctrl-b ,` |
| Next window | `Ctrl-b n` |
| Previous window | `Ctrl-b p` |
| List windows | `Ctrl-b w` |
| Vertical split | `Ctrl-b %` |
| Horizontal split | `Ctrl-b :` then `split-window` |

---

## The Prefix Key

All tmux commands start with the **prefix key**: `Ctrl-b`.

> Example: To create a new window, you press `Ctrl-b`, release, then press `c`.

