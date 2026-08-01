# Windows commands

Personal cheat sheet — quick lookup for Windows Command Prompt and the built-in admin tools. Adds grow as I learn more.

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
| `cls` | Clear the screen |

## Reading files

| Command | Use |
| --- | --- |
| `type <file>` | Print file contents to terminal (Windows equivalent of `cat`) |

## Getting help

| Command | Use |
| --- | --- |
| `<command> /?` | Built-in help for almost every cmd command |
| `net help <sub>` | Help for `net` sub-commands (uses `help`, not `/?`) |

## System info

| Command | Use |
| --- | --- |
| `whoami` | Current logged-in user |
| `whoami /groups` | Group memberships for current user |
| `whoami /priv` | Enabled/disabled privileges (`SeImpersonatePrivilege` etc.) |
| `hostname` | Computer name |
| `systeminfo` | OS name, version, system type, hotfixes |
| `msinfo32` | Big read-only dump: hardware, OS, drivers, network |
| `ver` | Windows version string |

## Networking

| Command | Use |
| --- | --- |
| `ipconfig` | IPv4 address, default gateway |
| `ipconfig /all` | MAC, DNS, DHCP, lease info |
| `ipconfig /release` / `/renew` | Drop / re-acquire DHCP lease |
| `ipconfig /flushdns` | Clear resolver cache |
| `netstat` | Active TCP connections |
| `netstat -a` | All listening + established sockets |
| `netstat -b` | Show owning executable per connection |
| `netstat -ano` | All + owning PID (cross-ref with tasklist) |
| `net share` | List shared folders (`C$`, `ADMIN$`, custom) |
| `net use` | Current mapped drives / SMB connections |
| `net use \\<host>\C$ /user:<u> <p>` | Mount remote admin share (lateral movement) |
| `arp -a` | Local ARP cache |

## Users and groups

| Command | Use |
| --- | --- |
| `net user` | List local users |
| `net user <name>` | Detail on one user |
| `net localgroup` | List local groups |
| `net localgroup Administrators` | Members of Administrators |
| `net localgroup Administrators <u> /add` | Add user to Administrators (requires admin) |

## Processes and services

| Command | Use |
| --- | --- |
| `tasklist` | List running processes |
| `tasklist /svc` | Include hosted services per svchost |
| `taskkill /PID <pid> /F` | Force-kill a process |
| `sc query` | List running services |
| `sc query state= all` | All services (note the space after `=`) |
| `sc qc <name>` | Config for a service (binpath, start type, account) |
| `sc start <name>` / `sc stop <name>` | Start / stop a service |

## Scheduled tasks

| Command | Use |
| --- | --- |
| `schtasks /query` | List scheduled tasks |
| `schtasks /query /fo LIST /v` | Verbose listing |
| `schtasks /create /tn <name> /tr <cmd> /sc onlogon /ru SYSTEM` | Create a task (persistence pattern) |
| `schtasks /delete /tn <name> /f` | Delete task |

## Registry (from cmd)

| Command | Use |
| --- | --- |
| `reg query <key>` | Read a key and its values |
| `reg add <key> /v <name> /t REG_SZ /d <data>` | Add / set a value |
| `reg delete <key> /v <name> /f` | Delete a value |
| `reg save <key> <file>` | Save a hive to disk (used offline to dump SAM / SYSTEM / SECURITY) |

**Common keys to know:**

| Key | Why |
| --- | --- |
| `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` | System-wide autostart programs |
| `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` | Per-user autostart programs |
| `HKLM\SYSTEM\CurrentControlSet\Services` | Every service definition |
| `HKLM\Software\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated` | If `1` → easy MSI privesc |

## Filesystem / permissions

| Command | Use |
| --- | --- |
| `icacls <path>` | Show NTFS ACL |
| `icacls <path> /grant <user>:F` | Grant Full control |
| `icacls <path> /remove <user>` | Remove all ACEs for a user |
| `attrib +h <file>` / `attrib -h <file>` | Hide / unhide file |
| `dir /r` | List Alternate Data Streams (NTFS ADS) |

## Environment variables

| Variable | Points to |
| --- | --- |
| `%windir%` | `C:\Windows` |
| `%SystemRoot%` | Same as `%windir%` |
| `%USERPROFILE%` | `C:\Users\<user>` |
| `%APPDATA%` | `C:\Users\<user>\AppData\Roaming` |
| `%LOCALAPPDATA%` | `C:\Users\<user>\AppData\Local` |
| `%TEMP%` / `%TMP%` | Temp folder (common malware staging) |
| `%PATH%` | Executable search path |
| `%COMPUTERNAME%` | Same as `hostname` |

Echo one with `echo %USERPROFILE%` — first thing after landing on a box.

## Run dialog (Win+R) — admin tools

| Command | Opens |
| --- | --- |
| `msconfig` | System Configuration (boot, services, tools) |
| `compmgmt.msc` | Computer Management (parent console) |
| `taskschd.msc` | Task Scheduler |
| `eventvwr.msc` | Event Viewer |
| `lusrmgr.msc` | Local Users and Groups |
| `services.msc` | Services |
| `devmgmt.msc` | Device Manager |
| `diskmgmt.msc` | Disk Management |
| `perfmon` | Performance Monitor |
| `resmon` | Resource Monitor |
| `msinfo32` | System Information |
| `regedit` | Registry Editor |
| `taskmgr` | Task Manager |
| `control` | Control Panel |
| `ms-settings:` | Settings app |

## Fast enumeration one-liners (post-compromise)

```cmd
whoami && whoami /groups && whoami /priv
net user && net localgroup Administrators
ipconfig /all && netstat -ano
net share
schtasks /query /fo LIST /v
sc query state= all
systeminfo
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

## Sources

- TryHackMe — Windows CLI Basics
- TryHackMe — Windows Fundamentals 1 (NTFS, ADS, %windir%, UAC, lusrmgr)
- TryHackMe — Windows Fundamentals 2 (msconfig, compmgmt.msc, Task Scheduler, Event Viewer, shares, resmon, regedit)
