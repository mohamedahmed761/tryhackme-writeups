# Linux commands

Personal cheat sheet — quick lookup for the Linux shell. Adds grow as I learn more.

## Navigation

| Command | Use |
| --- | --- |
| `pwd` | Print working directory (full path) |
| `ls` | List files and folders |
| `ls -l` | Long format: permissions, owner, size, date |
| `ls -a` | Include hidden items (anything starting with `.`) |
| `ls -al` / `ls -la` | Long format including hidden |
| `ls -lh` | Long format with human-readable file sizes |
| `cd <folder>` | Move into folder |
| `cd ..` | Move up one level |
| `cd ~` or `cd` | Jump to home directory |
| `cd /` | Jump to filesystem root |
| `find <path> -name <name>` | Recursive search by name from a starting point |
| `find -name "*.txt"` | Wildcard search within current directory tree |
| `find / -perm -4000 2>/dev/null` | Find SUID binaries (privesc lead) |
| `find / -writable -type f 2>/dev/null` | Find world-writable files |

## Reading files

| Command | Use |
| --- | --- |
| `cat <file>` | Print file contents to terminal |
| `file <file>` | Identify what a file actually is regardless of extension |
| `grep "<value>" <file>` | Search inside a file for a string |
| `grep -R "<value>" <path>` | Recursive search across a directory tree |
| `grep -i` | Case-insensitive |
| `wc -l <file>` | Count lines in a file |

## Creating and modifying files

| Command | Use |
| --- | --- |
| `touch <file>` | Create an empty file (or update its timestamp) |
| `mkdir <dir>` | Create a directory |
| `cp <src> <dst>` | Copy a file |
| `cp -r <src> <dst>` | Copy a directory recursively |
| `mv <src> <dst>` | Move or rename a file |
| `rm <file>` | Delete a file |
| `rm -R <dir>` | Delete a directory and everything inside it |

## Shell operators

| Operator | Effect |
| --- | --- |
| `&` | Run the command in the background |
| `&&` | Run next command only if the previous succeeded |
| `\||` | Run next command only if the previous failed |
| `>` | Redirect output to a file, overwriting existing contents |
| `>>` | Redirect output to a file, appending |
| `\|` (pipe) | Send output of one command as input to the next |

## System info

| Command | Use |
| --- | --- |
| `whoami` | Current logged-in user |
| `id` | UID, GID, and group memberships |
| `uname` | Kernel name (`Linux`) |
| `uname -a` | Full system info: kernel, hostname, kernel version, architecture, OS type |
| `df -h` | Disk space, human-readable sizes |
| `cat /etc/os-release` | Distro name, version, codename |
| `hostname` | Machine name |

## Users, groups, and permissions

| Command | Use |
| --- | --- |
| `su <user>` | Switch to another user (keep current dir) |
| `su -l <user>` | Switch and load the target user's login environment |
| `sudo <command>` | Run a single command as root |
| `sudo -l` | List commands the current user can run with sudo (first thing to run on a compromised box) |
| `chmod <mode> <file>` | Change permissions. `chmod 755 script.sh` |
| `chown <user>:<group> <file>` | Change owner and group |

**Numeric permissions:**
- `4` = read, `2` = write, `1` = execute
- Three digits: owner, group, other
- Common modes: `777` (everyone all), `755` (executable), `644` (document), `700` (owner only), `600` (SSH private key)

## Documentation

| Command | Use |
| --- | --- |
| `<command> --help` | Short usage summary |
| `man <command>` | Full manual page. `q` quits, `/` searches |
| `type <command>` | Show whether a command is a builtin, alias, or binary |
| `which <command>` | Show the path of the binary that runs |

## Remote access (SSH)

| Command | Use |
| --- | --- |
| `ssh <user>@<host>` | Connect to remote machine (password auth) |
| `ssh -i <key> <user>@<host>` | Connect using a private key |
| `ssh -p <port> <user>@<host>` | Non-default port |
| `scp <src> <user>@<host>:<dst>` | Copy file to remote machine |
| `scp <user>@<host>:<src> <dst>` | Copy file from remote machine |

**Password entry over SSH is invisible.** No dots or stars — the terminal is still reading, just keep typing.

## Common Linux directories

| Path | What lives there |
| --- | --- |
| `/etc` | System config. `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`, `/etc/os-release` |
| `/var` | Variable data. `/var/log` for all service logs |
| `/root` | Home directory of the root user |
| `/tmp` | Temporary storage. World-writable. Cleared on reboot |
| `/home` | Regular user home directories |
| `/usr` | Installed software. `/usr/share/wordlists` for SecLists / dirbuster |
| `/proc` | Kernel-exposed process and system state (virtual filesystem) |
| `/opt` | Third-party software installations |

## Quick offensive enumeration

One-liners worth memorising for CTFs / early privesc rooms.

```bash
# who am I and what can I do
whoami; id; sudo -l

# hunt SUID binaries
find / -perm -4000 -type f 2>/dev/null

# world-writable files
find / -perm -o+w -type f 2>/dev/null

# read user list without needing root
cat /etc/passwd

# hunt credentials in configs
grep -R "password" /etc/ 2>/dev/null

# check bash history for leaked commands
cat ~/.bash_history
```

## Sources

- [TryHackMe — Linux CLI Basics](https://tryhackme.com/room/linuxclibasics)
- [TryHackMe — Linux Fundamentals Part 1](https://tryhackme.com/room/linuxfundamentalspart1)
- [TryHackMe — Linux Fundamentals Part 2](https://tryhackme.com/room/linuxfundamentalspart2)
