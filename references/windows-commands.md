# Windows commands

Personal cheat sheet — quick lookup for Windows Command Prompt. Adds grow as I learn more.

## Navigation

| Command | Use |
| --- | --- |
| `cd` | Print current directory (no args) |
| `cd <folder>` | Move into folder |
| `cd ..` | Move up one level |
| `cd \` | Jump to drive root |
| `dir` | List files and folders |
| `dir /a` | List including hidden items |
| `dir /s <name>` | Recursive search by name from current directory |

## Reading files

| Command | Use |
| --- | --- |
| `type <file>` | Print file contents to terminal (Windows equivalent of `cat`) |

## System info

| Command | Use |
| --- | --- |
| `whoami` | Current logged-in user |
| `hostname` | Computer name |
| `systeminfo` | OS name, version, system type, and a lot more |
| `ipconfig` | Network config — IPv4 address, default gateway |
| `ipconfig /all` | Fuller network info (MAC address, DNS, DHCP) |

## Sources

- [TryHackMe — Windows CLI Basics](https://tryhackme.com/room/windowsclibasics)
