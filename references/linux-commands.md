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
| `zgrep` / `zcat` | Same as `grep` / `cat` for gzipped files (rotated logs) |
| `wc -l <file>` | Count lines in a file |
| `tail -f <file>` | Live-follow a file as it grows (log tailing) |

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

## Text editors

| Command | Use |
| --- | --- |
| `nano <file>` | Beginner-friendly modeless editor. `Ctrl+O` write, `Ctrl+X` exit, `Ctrl+W` search |
| `vim <file>` | Modal editor. `i` insert, `Esc` command mode, `:wq` save+quit, `:q!` force quit |

## Shell operators

| Operator | Effect |
| --- | --- |
| `&` | Run the command in the background |
| `&&` | Run next command only if the previous succeeded |
| `\|\|` | Run next command only if the previous failed |
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
| `chmod +x <file>` | Make a file executable |
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

## Remote access and file transfer

| Command | Use |
| --- | --- |
| `ssh <user>@<host>` | Connect to remote machine (password auth) |
| `ssh -i <key> <user>@<host>` | Connect using a private key |
| `ssh -p <port> <user>@<host>` | Non-default port |
| `scp <src> <user>@<host>:<dst>` | Copy file to remote machine |
| `scp <user>@<host>:<src> <dst>` | Copy file from remote machine |
| `scp -r <dir>` | Recursive — for directories |
| `wget <url>` | Download a file from HTTP/HTTPS |
| `curl -O <url>` | Same idea, comes preinstalled on more systems |
| `python3 -m http.server` | Instant HTTP server on port 8000 serving the current directory |
| `python3 -m http.server <port>` | Same, different port |

**Password entry over SSH is invisible.** No dots or stars — the terminal is still reading, just keep typing.

## Processes and jobs

| Command | Use |
| --- | --- |
| `ps` | List processes in your current session |
| `ps aux` | List every process on the system |
| `ps -ef` | Alternate style, full command line visible |
| `top` | Live process view. `q` to quit |
| `htop` | Nicer `top` (may need install) |
| `kill <PID>` | Send SIGTERM (polite shutdown) |
| `kill -9 <PID>` | Send SIGKILL (force stop, no cleanup) |
| `kill -STOP <PID>` | Pause a process |
| `kill -CONT <PID>` | Resume a paused process |
| `pkill <name>` | Kill by process name |
| `jobs` | List jobs backgrounded in this shell |
| `fg` | Bring backgrounded job to foreground |
| `bg` | Continue a suspended job in the background |
| `Ctrl+Z` | Suspend current foreground job |
| `nohup <cmd> &` | Run in background, survive logout |

**Signal reference:**
- `SIGTERM` (15) — polite shutdown request, catchable
- `SIGKILL` (9) — immediate kill, uncatchable
- `SIGSTOP` (19) — pause
- `SIGCONT` — resume
- `SIGHUP` (1) — terminal hangup; many daemons reload config on this

## Services (systemd)

| Command | Use |
| --- | --- |
| `systemctl start <service>` | Start now |
| `systemctl stop <service>` | Stop now |
| `systemctl restart <service>` | Stop then start |
| `systemctl status <service>` | Current state + last log lines |
| `systemctl enable <service>` | Auto-start on boot |
| `systemctl disable <service>` | Don't auto-start on boot |
| `systemctl list-units --type=service` | List loaded services |
| `journalctl -u <service>` | Read log for a systemd service |
| `journalctl -f` | Live-tail the system journal |

**Unit files:** `/etc/systemd/system/*.service` (custom / third-party), `/lib/systemd/system/*.service` (distro-provided).

## Scheduled tasks (cron)

| Command | Use |
| --- | --- |
| `crontab -e` | Edit current user's crontab |
| `crontab -l` | List current user's crontab |
| `crontab -r` | Remove current user's crontab |
| `sudo crontab -u <user> -e` | Edit another user's crontab |

**Crontab format:** `MIN HOUR DOM MON DOW CMD`

```
# every day at 3am
0 3 * * * /home/user/backup.sh

# every 12 hours
0 */12 * * * cp -R /home/user/Documents /var/backups/

# on boot
@reboot /home/user/startup.sh
```

**Shortcuts:** `@reboot`, `@yearly`, `@monthly`, `@weekly`, `@daily`, `@hourly`.

**System-wide locations:**
- `/etc/crontab` — main system crontab
- `/etc/cron.d/` — drop custom crontab files here
- `/etc/cron.daily/` / `hourly` / `weekly` / `monthly` — drop scripts, they run on that cadence

Check ALL of these when investigating persistence.

## Package management (apt)

| Command | Use |
| --- | --- |
| `sudo apt update` | Refresh available-package list |
| `sudo apt upgrade` | Upgrade all installed packages |
| `sudo apt install <pkg>` | Install a package |
| `sudo apt remove <pkg>` | Uninstall (leaves config files) |
| `sudo apt purge <pkg>` | Uninstall + remove config |
| `sudo apt autoremove` | Clean up orphaned dependencies |
| `apt search <keyword>` | Search available packages |
| `apt show <pkg>` | Package details |
| `apt list --installed` | Everything installed |
| `sudo add-apt-repository ppa:<user>/<ppa>` | Add a third-party PPA |
| `sudo dpkg -i <file.deb>` | Install a .deb package directly |
| `dpkg -l` | List all installed packages (dpkg-level) |

**Repository files:** `/etc/apt/sources.list` and `/etc/apt/sources.list.d/*.list`.
**GPG keys:** `/etc/apt/trusted.gpg.d/`.

## Logs

| Path | What's inside |
| --- | --- |
| `/var/log/auth.log` | Login attempts, sudo use, SSH connections |
| `/var/log/syslog` | General system messages |
| `/var/log/kern.log` | Kernel messages |
| `/var/log/dpkg.log` | Package install / remove history |
| `/var/log/apache2/access.log` | Every HTTP request |
| `/var/log/apache2/error.log` | Web server errors |
| `/var/log/nginx/access.log` | Same for nginx |
| `/var/log/fail2ban.log` | Brute-force blocks |
| `/var/log/ufw.log` | Firewall events |

**Rotated logs.** Old logs get gzipped (`auth.log.1.gz`, `auth.log.2.gz`). Search inside with `zgrep "pattern" /var/log/auth.log*`.

## Common Linux directories

| Path | What lives there |
| --- | --- |
| `/etc` | System config. `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`, `/etc/os-release` |
| `/var` | Variable data. `/var/log`, `/var/backups`, `/var/www` |
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

# find running services
ps aux
systemctl list-units --type=service --state=running

# check scheduled jobs — all the places
crontab -l
cat /etc/crontab
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/ /etc/cron.weekly/

# read recent auth events
sudo tail -n 100 /var/log/auth.log
zgrep "Failed password" /var/log/auth.log*
```

## Sources

- [TryHackMe — Linux CLI Basics](https://tryhackme.com/room/linuxclibasics)
- [TryHackMe — Linux Fundamentals Part 1](https://tryhackme.com/room/linuxfundamentalspart1)
- [TryHackMe — Linux Fundamentals Part 2](https://tryhackme.com/room/linuxfundamentalspart2)
- [TryHackMe — Linux Fundamentals Part 3](https://tryhackme.com/room/linuxfundamentalspart3)
