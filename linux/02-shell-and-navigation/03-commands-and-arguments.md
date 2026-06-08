# Commands and Arguments

You've seen commands like `ls`, `cd`, and `pwd`. But how do Linux commands actually work? What's the structure behind them?

Understanding this structure will make **every** command you learn from here on click faster.

---

## The Anatomy of a Command

Every Linux command follows this general pattern:

```
command   [options]   [arguments]
```

| Part | What It Is | Required? |
|------|-----------|-----------|
| **Command** | The program to run | ✅ Always |
| **Options** (flags) | Modify *how* the command behaves | ❌ Optional |
| **Arguments** | What the command acts *on* (files, directories, text) | ❌ Usually optional |

### Let's Break Down Real Examples

```bash
ls -l /home
│   │    │
│   │    └── Argument: "show me the contents of /home"
│   └─────── Option: "-l" (long format)
└─────────── Command: "list directory contents"
```

```bash
cp -r /var/log /backup
│  │    │        │
│  │    │        └── Argument 2: destination
│  │    └─────────── Argument 1: source
│  └──────────────── Option: "-r" (recursive — copy folders too)
└─────────────────── Command: "copy"
```

```bash
grep -i "error" /var/log/syslog
│    │    │         │
│    │    │         └── Argument 2: file to search in
│    │    └──────────── Argument 1: pattern to search for
│    └───────────────── Option: "-i" (case-insensitive)
└────────────────────── Command: "search for text"
```

---

## Options (Flags) — The Details

Options modify how a command works. They come in two forms:

### Short Options (Single Dash)

Single letter, prefixed with `-`:

```bash
ls -l          # long format
ls -a          # show hidden files
```

**Combining short options** — these are equivalent:

```bash
ls -l -a -h    # separate
ls -lah        # combined — same thing, shorter to type
```

### Long Options (Double Dash)

Full words, prefixed with `--`:

```bash
ls --all                  # same as -a
grep --ignore-case        # same as -i
cp --recursive            # same as -r
```
---

## Arguments — What the Command Acts On

Arguments tell the command **what** to operate on:

```bash
# No argument — operates on current directory by default
ls

# One argument — operates on the specified directory
ls /home

# Multiple arguments — operates on each
ls /home /var /tmp

# Arguments can be files too
cat /etc/hostname
head -5 /var/log/syslog
```

### Commands That Don't Need Arguments

Some commands work fine with no arguments — they have sensible defaults:

```bash
pwd         # prints current directory — no argument needed
whoami      # prints your username — no argument needed
date        # prints current date/time — no argument needed
ls          # lists current directory — default is "."
```

### Commands That Require Arguments

Some commands make no sense without arguments:

```bash
cd /var/log        # needs to know WHERE to go
cp file1 file2     # needs to know WHAT to copy and WHERE
mv old.txt new.txt # needs to know WHAT to rename
mkdir projects     # needs to know WHAT to create
```

---

## Getting Help — How to Learn Any Command

You don't need to memorize every option. Linux has built-in help systems:

### 1. The `--help` Flag

Almost every command supports `--help`:

```bash
ls --help
cp --help
grep --help
```

This gives you a quick summary of all available options.

### 2. The `man` Command (Manual Pages)

For detailed documentation:

```bash
man ls      # opens the full manual for ls
man cp      # opens the full manual for cp
man grep    # opens the full manual for grep
```

---

## Command Types — Not Everything Is a Program

When you type a command, it could be one of several things:

| Type | What It Is | Example |
|------|-----------|---------|
| **Executable** | A program file on disk | `ls`, `grep`, `docker`, `nginx` |
| **Shell Builtin** | Built into the shell itself (no separate program) | `cd`, `echo`, `pwd`, `export` |
| **Alias** | A shortcut for a longer command | `ll` → `ls -la` |
| **Function** | A shell function defined by the user | Custom functions in `.bashrc` |

```bash
# See what something is
type ls       # /usr/bin/ls (executable)
type cd       # builtin
type ll       # alias for 'ls -la'
```

---

## Putting It All Together — Real Examples

```bash
# Simple: no options, no arguments
pwd
whoami
date

# With options
ls -l
ps aux
df -h

# With arguments
cat /etc/os-release
mkdir my-project

# With options AND arguments
grep -r "TODO" /home/adithya/projects/
find /var/log -name "*.log" -mtime -7
tar -czf backup.tar.gz /home/adithya/
```

Reading it back:

| Command | Translation |
|---------|-------------|
| `grep -r "TODO" /projects/` | Search **recursively** for "TODO" in the /projects directory |
| `find /var/log -name "*.log" -mtime -7` | Find files in /var/log **named** *.log, **modified** in the last 7 days |
| `tar -czf backup.tar.gz /home/` | **Create** a g**z**ipped tar **f**ile called backup.tar.gz from /home |

---

## Key Takeaways

- Every command follows: **`command [options] [arguments]`**.
- **Options** modify behavior: short (`-l`) for typing, long (`--all`) for scripts.
- Short options can be **combined**: `-l -a -h` → `-lah`.
- **Arguments** tell the command what to act on — files, directories, text.
- Use `--help`, `man`, and `type` to learn about any command — **no memorization needed**.
- Not all commands are programs — some are **builtins**, **aliases**, or **functions**.

---

**← Previous:** [Home Directory](./02-home-directory.md) · **Next →** [Relative Path vs Absolute Path](./04-relative-vs-absolute-path.md)
