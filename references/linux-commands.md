# Linux commands

Personal cheat sheet — quick lookup for the Linux shell. Adds grow as I learn more.

## Navigation

| Command | Use |
| --- | --- |
| `pwd` | Print working directory (full path) |
| `ls` | List files and folders |
| `ls -l` | Long format: permissions, owner, size, date |
| `ls -a` | Include hidden items (anything starting with `.`) |
| `ls -al` | Long format including hidden |
| `cd <folder>` | Move into folder |
| `cd ..` | Move up one level |
| `cd ~` or `cd` | Jump to home directory |
| `cd /` | Jump to filesystem root |
| `find <path> -name <name>` | Recursive search by name from a starting point |

## Reading files

| Command | Use |
| --- | --- |
| `cat <file>` | Print file contents to terminal |

## System info

| Command | Use |
| --- | --- |
| `whoami` | Current logged-in user |
| `uname` | Kernel name (`Linux`) |
| `uname -a` | Full system info: kernel, hostname, kernel version, architecture, OS type |
| `df -h` | Disk space, human-readable sizes |
| `cat /etc/os-release` | Distro name, version, codename |

## Sources

- [TryHackMe — Linux CLI Basics](https://tryhackme.com/room/linuxclibasics)
