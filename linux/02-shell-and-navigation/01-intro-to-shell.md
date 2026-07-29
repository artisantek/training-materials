# Introduction to the Shell / Terminal

When you connect to a Linux server — say, an EC2 instance via SSH — there's no desktop, no icons, no mouse. You get a **blinking cursor on a black screen**. That's the **shell**.

It might look intimidating at first, but the shell is the single most powerful tool you'll use as a DevOps engineer. Let's understand what it actually is.

---

## What is the Shell?

The **shell** is a program that takes commands you type and passes them to the operating system to execute. It's the **middleman** between you and the Linux kernel.

<div align="center">
  <img src="../images/shell-kernel-hardware-flow.png" alt="Shell to Kernel to Hardware Flow" width="450" />
</div>

When you type `ls` and press Enter:
1. The **shell** reads your input
2. It finds the `ls` program on disk
3. It asks the **kernel** to run it
4. The kernel talks to the **hardware** (disk) to list files
5. The result comes back through the kernel → shell → your screen

---

## Shell vs Terminal — Are They the Same?

People often use these terms interchangeably, but they're technically different:

| Term | What It Is | Analogy |
|------|-----------|---------|
| **Terminal** | The **window/application** that displays text and accepts keyboard input | The **TV screen** |
| **Shell** | The **program running inside** the terminal that interprets commands | The **TV channel** — the actual content |

- On your **local machine** (Mac/Windows), the terminal is an app — Terminal.app, iTerm2, Windows Terminal, etc.
- On a **remote server** (via SSH), your local terminal connects to the server's shell remotely.

You can change the shell without changing the terminal, and vice versa.

---

## The Evolution of Shells

Linux and Unix support multiple shells. Over the years, they have evolved to add more features. Here is the brief history you should know:

| Year | Shell | Full Name | What Changed? |
|------|-------|-----------|---------------|
| **1979** | `sh` | **Bourne Shell** | The original Unix shell created by Stephen Bourne. Very minimal. Scripts written in `sh` are highly portable and run everywhere. |
| **1978** | `csh` | **C Shell** | Created by Bill Joy. Added command history and a syntax similar to the C programming language. (Later improved as `tcsh`). |
| **1983** | `ksh` | **KornShell** | Combined the best features of `sh` and `csh`. Very popular in enterprise Unix environments. |
| **1989** | `bash` | **Bourne Again Shell** | An open-source clone of `sh` with features from `ksh` and `csh`. **This is the default on almost all Linux servers today** (Ubuntu, RHEL, Amazon Linux). |
| **1990** | `zsh` | **Z Shell** | Extremely feature-rich with advanced autocompletion and theming. It is now the **default on macOS**. |

> 💡 **For DevOps work, `bash` is the undisputed standard.** When you write automation scripts for servers, you will almost always use `bash`.

### How to Check Your Current Shell

```bash
# Check the SHELL environment variable
echo $SHELL
# Output: /bin/bash
```

---

## The Shebang (`#!`)

Because there are so many different shells (`sh`, `bash`, `zsh`), how does Linux know *which* shell to use when executing a script? 

This is where the **Shebang** comes in.

If you write a shell script, the very first line should look like this:

```bash
#!/bin/bash

echo "Hello, DevOps!"
```

### Why is it needed?
The `#!` (pronounced "shebang" or "hashbang") tells the kernel: *"Please use the program located at `/bin/bash` to run this file."*

- If you want a script to run using the old, highly portable `sh`, you use `#!/bin/sh`.
- If you write a Python script, you can even use `#!/usr/bin/env python3`.

### What happens if you don't use it?
If you forget the shebang, Linux will attempt to run the script using your **current active shell**. 
- If you are using `bash`, it runs in `bash`.
- If another user runs it while using `zsh`, it runs in `zsh`. 

This is dangerous because a script that works perfectly in `bash` might fail or behave differently in `zsh` or `sh`. **Always use a shebang** to guarantee your script runs exactly how you intended, no matter who runs it.

---

## The Shell Prompt — Reading It

When you log in, you see something like this:

```
adithya@linux:~$
```

Every part of this prompt tells you something:

```
adithya  @  linux  :  ~       $
  │      │    │    │  │       │
  │      │    │    │  │       └── $ = regular user (# = root user)
  │      │    │    │  └────────── ~ = current directory (home)
  │      │    │    └───────────── separator
  │      │    └────────────────── hostname (server name)
  │      └─────────────────────── at
  └────────────────────────────── username
```

### Key Things to Notice

| Prompt Ending | Meaning |
|---------------|---------|
| `$` | You're a **regular user** |
| `#` | You're the **root user** (admin) — be careful! |

| Directory Shown | Meaning |
|-----------------|---------|
| `~` | You're in your **home directory** |
| `/etc` | You're in the `/etc` directory |
| `/` | You're at the **root** of the filesystem |

So when you see:
```
root@production-server:/etc#
```
You know: you're logged in as **root**, on a server called **production-server**, in the `/etc` directory.

---

### Where Does the Shell Look for Commands?

The shell uses an environment variable called **PATH** to know where to find programs:

```bash
echo $PATH
# Output: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

This is a **colon-separated list of directories**. When you type `ls`, the shell searches these directories in order until it finds an executable called `ls`.

```bash
# Find out exactly where a command lives
which ls
# Output: /usr/bin/ls

which docker
# Output: /usr/bin/docker
```

---

## Essential Shell Shortcuts

These will make you dramatically faster on the terminal:

### Navigation

| Shortcut | Action |
|----------|--------|
| `Ctrl + A` | Jump to the **beginning** of the line |
| `Ctrl + E` | Jump to the **end** of the line |
| `Ctrl + ←/→` | Jump **word by word** |

### Editing

| Shortcut | Action |
|----------|--------|
| `Ctrl + U` | Delete everything **before** the cursor |
| `Ctrl + K` | Delete everything **after** the cursor |
| `Ctrl + W` | Delete the **word** before the cursor |

### History

| Shortcut | Action |
|----------|--------|
| `↑ / ↓` | Scroll through **previous commands** |
| `Ctrl + R` | **Reverse search** — type to search your command history |
| `history` | Show your full command history |
| `!!` | Repeat the **last command** |
| `!docker` | Run the **last command that started with** "docker" |

### Session

| Shortcut | Action |
|----------|--------|
| `Ctrl + C` | **Cancel** the currently running command |
| `Ctrl + D` | **Exit** the shell (logout) |
| `Ctrl + L` | **Clear** the screen (same as `clear` command) |

> 💡 **Pro tip:** `Ctrl + R` is a game-changer. Instead of pressing ↑ fifty times, just hit `Ctrl + R` and type a few letters of the command you're looking for.

---

## Key Takeaways

- The **shell** is a program that interprets your commands and talks to the kernel. The **terminal** is the window that displays the shell.
- **bash** is the default shell on most Linux servers — it's the one you need to know.
- The **prompt** tells you your username, hostname, current directory, and whether you're root or a regular user.
- Every command goes through a **read → parse → search → execute → output** cycle.
- Learn the **keyboard shortcuts** — they'll save you hours over time.

---

**Next →** [Home Directory](./02-home-directory.md)
