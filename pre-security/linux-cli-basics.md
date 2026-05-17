# Linux CLI Basics

**Path:** Pre-Security → Operating Systems Basics
**Room:** [Linux CLI Basics](https://tryhackme.com/room/linuxclibasics)
**Difficulty:** Easy · **Time:** ~45 min
**Completed:** 2026-05-17

## What the room covered

First real practice with the Linux terminal. Two missions on a target machine: navigate the filesystem to find a hidden brief (`mission_brief.txt`), then run system info commands to fill out a short report. Ended with a self-guided mini challenge to find and read `day1_report.txt` without step-by-step instructions.

The four tasks: intro, Navigation Mission (pwd / ls / cd / find / cat), Investigating the System (whoami / uname / df / reading /etc/os-release), and conclusion.

## Key concepts

**Where am I, what's around me, how do I move.**
- `pwd` — print working directory. Shows the absolute path of the folder you're sitting in.
- `ls` — list contents. `ls -l` adds permissions, owner, size, date. `ls -al` also shows hidden files (anything starting with `.`).
- `cd <dir>` — change directory. `cd ..` goes up one level. `~` means your home directory.

**Finding and reading files.**
- `find <starting_point> -name <filename>` — walks every folder under the starting point looking for a name match. `find ~ -name mission_brief.txt` searches your whole home directory.
- `cat <file>` — dumps the file's contents to the terminal.

**System info.**
- `whoami` — current logged-in username.
- `uname -a` — everything about the system: kernel name (`Linux`), hostname, kernel version, architecture (`x86_64`), OS type (`GNU/Linux`). `uname` alone just prints the kernel name.
- `df -h` — disk space in human-readable sizes (G, M) instead of raw bytes. Shows the real disk (`/dev/root`) plus several `tmpfs` entries that live in RAM, not on disk.
- `/etc/os-release` — plain text file every Linux distro has, with clearer distribution info than `uname` (PRETTY_NAME, VERSION, codename, etc.).

**Hidden files aren't really hidden.** Anything starting with a dot is just convention — `ls -a` shows them.

**Workflow that ties it together.** `find` to locate → `cd` into the folder → `ls` to confirm → `cat` to read. The mini challenge at the end was literally this loop with no hand-holding.

## What clicked

The find → cd → cat chain. Up to that point I was treating each command as a separate trivia card. Running `find ~ -name mission_brief.txt` and watching it print the full path, then pasting that into `cd`, then `cat` to read the file — that was the first time the CLI felt like one tool instead of a list of words to memorize. The mini challenge at the end with `day1_report.txt` proved it stuck because I could do it without re-reading the steps.

## What to revisit

Reading `ls -l` output. I can run the command, but the columns (`drwxr-xr-x 2 ubuntu ubuntu 4096 Feb 27 2022 Desktop`) still look like noise — I want to be able to glance at the permission string and know what it means (file vs directory, what owner/group/everyone can do) without thinking. Same with the `df -h` output — I get the gist (size / used / available / mount point) but I want to actually understand what `tmpfs` is and why there are several of them.
