# Linux CLI Basics

**Path:** Pre-Security → Operating Systems Basics
**Room:** [Linux CLI Basics](https://tryhackme.com/room/linuxclibasics)
**Difficulty:** Easy · **Time:** ~45 min · **Completed:** 2026-05-17

## Mission 1 — Find the brief

Where am I, what's here.

```bash
ubuntu@tryhackme:~$ pwd
/home/ubuntu

ubuntu@tryhackme:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos  logs  projects  snap

ubuntu@tryhackme:~$ ls -l
total 44
drwxr-xr-x 2 ubuntu ubuntu 4096 Feb 27  2022 Desktop
drwxr-xr-x 6 ubuntu ubuntu 4096 Dec 11 12:45 Documents
drwxr-xr-x 2 ubuntu ubuntu 4096 Feb 16  2024 Downloads
... one row per item: perms, owner, group, size, date, name
```

Move around.

```bash
ubuntu@tryhackme:~$ cd Documents
ubuntu@tryhackme:~/Documents$ pwd
/home/ubuntu/Documents
ubuntu@tryhackme:~/Documents$ cd ..
ubuntu@tryhackme:~$ pwd
/home/ubuntu
```

`cd <dir>` enters. `cd ..` goes up. `~` is home.

Find the file.

```bash
ubuntu@tryhackme:~$ find ~ -name mission_brief.txt
/home/ubuntu/Documents/.research/archive/mission_brief.txt
```

The folder starts with a dot, so `ls` hides it. `ls -a` would show it.

Open it.

```bash
ubuntu@tryhackme:~$ cd /home/ubuntu/Documents/.research/archive
ubuntu@tryhackme:~/Documents/.research/archive$ ls
mission_brief.txt
ubuntu@tryhackme:~/Documents/.research/archive$ cat mission_brief.txt
Great job finding your way around the terminal.

Your next assignment is to collect a small system report:
- Who you're logged in as
- The kernel version
- Total disk space
- The name of this Linux distribution

FLAG:<redacted>
```

## Mission 2 — System report

```bash
ubuntu@tryhackme:~$ whoami
ubuntu

ubuntu@tryhackme:~$ uname -a
Linux tryhackme 6.14.0-1018-aws #18~24.04.1-Ubuntu SMP Mon Nov 24 19:46:27 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

`uname -a` breakdown: kernel name · hostname · kernel version · architecture · OS type.

```bash
ubuntu@tryhackme:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        68G   11G   58G  16% /
tmpfs           969M     0  969M   0% /dev/shm
tmpfs           388M  1.2M  387M   1% /run
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           194M  188K  194M   1% /run/user/1000
tmpfs           194M  172K  194M   1% /run/user/114
```

`/dev/root` is the real disk. Everything `tmpfs` lives in RAM.

```bash
ubuntu@tryhackme:~$ cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.1 LTS (Noble Numbat)"
ID=ubuntu
ID_LIKE=debian
```

Cleaner distro info than `uname`.

## Commands learned

| Command | Job |
| --- | --- |
| `pwd` | Where am I |
| `ls` / `ls -l` / `ls -a` | What's here / details / hidden |
| `cd <dir>` / `cd ..` / `cd ~` | Move in / up / home |
| `find <path> -name <file>` | Search by name from a starting point |
| `cat <file>` | Print file to terminal |
| `whoami` | Current user |
| `uname -a` | Kernel and system info |
| `df -h` | Disk space, human-readable |

## What clicked

`find` → `cd` → `cat`. First time the CLI felt like one tool instead of a list of commands.

## What to revisit

`ls -l` permission strings (`drwxr-xr-x`) and what `tmpfs` actually is in `df -h`.
