# Linux Fundamentals Part 2

**Path:** Cyber Security 101 → Linux Fundamentals
**Room:** [Linux Fundamentals Part 2](https://tryhackme.com/room/linuxfundamentalspart2)
**Difficulty:** Info · **Time:** ~20 min · **Completed:** 2026-06-15

> Builds on [linux-fundamentals-1.md](./linux-fundamentals-1.md). See [linux-commands.md](../references/linux-commands.md) for the cheat sheet.

## What the room covered

How to actually connect to a remote Linux machine over SSH (the same protocol behind the in-browser terminal from Part 1), how to read command documentation (`--help` and `man`), and the next big batch of file and permission commands: `touch`, `mkdir`, `cp`, `mv`, `rm`, `file`, `su`, plus the numeric permission model (`755`, `644`, `chmod`) and the standard Linux directory layout.

## Key concepts

**SSH — the way you actually reach Linux boxes.**

```bash
ssh <user>@<ip>
# example
ssh tryhackme@10.10.123.45
```

Everything after the login runs on the remote machine. Password input is invisible (no dots, no stars) — that's normal, keep typing. SSH is encrypted end-to-end, so credentials and commands aren't sniffable in transit.

**Flags, switches, and `--help`.** Most commands take arguments prefixed with `-` (short) or `--` (long). `ls` alone shows visible files; `ls -a` shows hidden ones; `ls -lh` shows a long, human-readable listing.

Every serious command has a `--help` flag that dumps its options. For deeper documentation, use the manual page:

```bash
man ls        # full man page for ls
ls --help     # short usage summary
```

Inside `man`: **down arrow** scrolls line by line, **space** pages, **q** quits, **/** searches. Every Linux command you'll ever use has a man page — the ability to read one is more valuable than memorising flags.

**Filesystem interaction, part 2.**

| Command | What it does |
| --- | --- |
| `touch <file>` | Create an empty file. Also updates the timestamp if the file exists. |
| `mkdir <dir>` | Create a directory. |
| `cp <src> <dst>` | Copy a file. |
| `mv <src> <dst>` | Move or rename a file. |
| `rm <file>` | Delete a file. |
| `rm -R <dir>` | Delete a directory and everything inside it (recursive). |
| `file <file>` | Identify what a file actually is, regardless of its extension. |

`file` reads the first bytes and tells you "ASCII text", "ELF 64-bit executable", "PNG image", etc. Extensions lie; `file` tells the truth.

**Permissions — the `rwxrwxrwx` model.**

Every file has three permission groups: **owner**, **group**, **other**. Each group has three bits: **read** (`r`), **write** (`w`), **execute** (`x`).

`ls -l` shows the string:

```
-rw-r--r-- 1 cmnatic cmnatic 0 Feb 19 10:37 file1
```

Read as: owner can read/write, group can read, other can read.

**Numeric form.** Each bit has a value — read=4, write=2, execute=1 — and each group is the sum.

| Symbolic | Numeric | Meaning |
| --- | --- | --- |
| `rwxrwxrwx` | `777` | Everyone can do everything. Almost always wrong. |
| `rwxr-xr-x` | `755` | Owner full, group and other read/execute. Typical for executables. |
| `rw-r--r--` | `644` | Owner read/write, group and other read only. Typical for documents. |
| `rwx------` | `700` | Owner only. Correct for SSH private keys. |

Change them with `chmod`:

```bash
chmod 755 script.sh    # rwxr-xr-x
chmod 600 id_rsa       # owner read/write only
```

**Switching users.**

```bash
su <user>         # switch, keep current directory
su -l <user>      # "login shell" — drop into that user's home dir, load their env
```

You need the target user's password (unless you're root, which needs no password to switch).

**Common directories — the Linux root layout.**

| Directory | What lives there |
| --- | --- |
| `/etc` | System configuration files. `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`. |
| `/var` | Variable data — anything that changes at runtime. `/var/log` is where every service dumps its logs. |
| `/root` | Home directory of the root user. |
| `/tmp` | Temporary files. Cleared on reboot. World-writable. |
| `/home` | Home directories of regular users (`/home/tryhackme` etc.). |
| `/usr` | Installed software and shared resources. `/usr/share/wordlists` for CTFs. |

## Mental model

Part 1 taught you to navigate; Part 2 teaches you to **change things**. You can now create files, move them, delete them, restrict access to them, and switch identity to access what your current user can't. Combined with SSH, this is enough to actually operate on a real box.

A useful frame: every file has an owner, a group, three permission bits, and a location. If any one of those is wrong, that's an attack surface. Weak owner? Wrong group? Missing execute bit? World-writable in `/tmp`? Each one is either a bug or a lead.

## Offensive angle

| Concept | Attacker use |
| --- | --- |
| `ssh user@host` | Post-exploitation pivot. Weak SSH creds are the classic initial-access technique — Hydra + dictionary + exposed SSH port = shell. |
| `man <command>` | Read the target's docs. Version-specific behaviours and undocumented flags matter for exploits. |
| `touch` | Anti-forensics — `touch -t` sets a file timestamp, letting attackers hide when a file was really created or modified. |
| `rm -R` | Log cleanup. `rm /var/log/auth.log` erases the SSH login trail (if the attacker has the perms). |
| `cp` / `mv` | Stage payloads to `/tmp` (world-writable), execute, then `mv` them out to bury the trail. |
| `file <binary>` | On a machine you've compromised, verify a suspicious binary before running it. On a target, identify what actually runs from an obscure path. |
| `ls -la` on `/etc` | Look for the same-modified-time cluster — that's when someone last tampered with the box. |
| `/etc/passwd` | Enumerate every user. Anyone can read it by default. Cross-reference with `/etc/shadow` (root-only) for hash-cracking targets. |
| `/etc/shadow` | If readable by you as a non-root user — misconfiguration — game over. Dump it and crack offline with Hashcat / John. |
| `/etc/sudoers` | Who can escalate to root and with which commands. `sudo -l` is the first thing to run after landing on a shell. |
| Weak permissions | `find / -perm -o+w -type f 2>/dev/null` finds world-writable files. Overwriting the wrong one — a root cron script, a startup file — owns the box. |
| SUID binaries | `find / -perm -4000 2>/dev/null`. Binaries that run as their owner regardless of who executes them. GTFOBins is the reference for which SUID binaries can be abused to root. |
| `/tmp` | Everyone can write here. Standard place to drop enumeration scripts, stage payloads, or hide files (`/tmp/.something` — hidden by default). |
| `su` | Once you have credentials for another user (leaked history, weak passwords), `su` pivots you between accounts without re-authenticating over SSH. |
| `chmod 777` in scripts | Sometimes admins fix broken things by loosening permissions. That leaves world-writable configs, keys, and scripts you can exploit later. |

**The privilege-escalation loop.** Land on the box → `whoami`, `id`, `sudo -l` → look for SUID binaries, world-writable files, and misconfigured cron jobs → chain what you find into root. Every command in this room shows up in that loop.

## Practical — the room's chained workflow

```bash
# connect from the AttackBox to the lab machine
user@attackbox$ ssh tryhackme@<MACHINE_IP>
tryhackme@<MACHINE_IP>'s password: tryhackme

# read documentation before you touch anything
tryhackme@linux2:~$ man ls
tryhackme@linux2:~$ ls -lh

# create, copy, rename, delete
tryhackme@linux2:~$ touch note
tryhackme@linux2:~$ mkdir mydirectory
tryhackme@linux2:~$ cp note note2
tryhackme@linux2:~$ mv note2 note3
tryhackme@linux2:~$ rm note3
tryhackme@linux2:~$ rm -R mydirectory

# identify what a file actually is
tryhackme@linux2:~$ file unknown1

# switch to another user and read what only they can see
tryhackme@linux2:~$ su -l user2
Password: user2
user2@linux2:~$ cat important
```

## What clicked

The permission numbers stopped looking arbitrary. `755` isn't a random string — it's just `rwx r-x r-x` with each triplet summed. Once I saw that read=4, write=2, execute=1, and each group is the sum, every `chmod` command in every Linux tutorial suddenly parsed. The same logic explains why `600` is the correct mode for an SSH private key (only owner reads and writes; nobody else can touch it) and why SSH refuses to use a key that's `644`.

## What to revisit

- `sudo` and `sudo -l` — the room barely touches sudo, but reading and abusing sudoers is the first move in every Linux privesc room.
- `chown` and `chgrp` — changing ownership, not just permissions. Matters when copying files between users.
- Special permission bits — setuid (`4000`), setgid (`2000`), sticky bit (`1000`). The room mentions SUID by example (`-rwx-`); the theory is the missing piece before Linux PrivEsc.
- `/proc` and `/sys` — not in the common-directories list, but they're where the kernel exposes process and hardware state. Essential for real forensics.
