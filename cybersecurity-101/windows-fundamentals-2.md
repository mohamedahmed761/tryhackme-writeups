# Windows Fundamentals 2

**Path:** Cyber Security 101 → Windows and AD Fundamentals
**Room:** [Windows Fundamentals 2](https://tryhackme.com/room/windowsfundamentals2x0x)
**Difficulty:** Info · **Time:** ~30 min · **Completed:** 2026-07-21

> Builds on [windows-fundamentals-1.md](./windows-fundamentals-1.md). See [windows-commands.md](../references/windows-commands.md) for the cheat sheet.

## What the room covered

A tour of the built-in Windows admin surfaces beyond the basics: System Configuration (`msconfig`), Advanced System Settings (page file, crash dumps, UAC slider), Computer Management (`compmgmt.msc` — Task Scheduler, Event Viewer, Shared Folders, Services, WMI), System Information (`msinfo32`), Resource Monitor (`resmon`), Command Prompt basics, and the Windows Registry / Registry Editor (`regedit`).

## Key concepts

**System Configuration (`msconfig.exe`).** Advanced troubleshooting tool for startup issues. Five tabs:

| Tab | Purpose |
| --- | --- |
| General | Choose boot mode: Normal / Diagnostic / Selective |
| Boot | Boot options, safe boot, advanced boot parameters |
| Services | List every service, running or stopped |
| Startup | (deprecated on modern Windows; use Task Manager → Startup instead) |
| Tools | One-click launchers for other admin tools (event viewer, perfmon, etc.) with the exact command shown |

**Advanced System Settings.** Search **View advanced system settings** → System Properties. Three important bits here:

- **Page file** — disk-backed virtual memory that kicks in when RAM fills up. Set under Performance → Advanced.
- **Startup and Recovery** — what type of crash dump gets written on a BSOD. Options: Automatic memory dump / Kernel memory dump / Small memory dump (256 KB) / Complete memory dump / None. Analysts read these to diagnose blue screens or find lingering malware in memory.
- **Environment variables** — system-wide and per-user path/setting variables (`%windir%`, `%APPDATA%`, `%TEMP%`, `PATH`).

**Computer Management (`compmgmt.msc`).** The main admin console. Three sections:

**1. System Tools:**

| Tool | What it does |
| --- | --- |
| Task Scheduler | Create, list, and manage scheduled tasks (Windows' equivalent of cron). Tasks can run at boot, at login, on a schedule, or on an event. |
| Event Viewer | Windows' audit log. Split into Windows Logs (Application / Security / Setup / System / Forwarded) and application-specific logs. |
| Shared Folders | Every folder shared on this machine, plus who's currently connected. Includes default admin shares like `C$` and `ADMIN$`. |
| Local Users and Groups | Same `lusrmgr.msc` from WF1. |
| Performance | `perfmon` — live and logged performance counters. |
| Device Manager | Hardware inventory + driver management. |

**2. Storage:**

| Tool | What it does |
| --- | --- |
| Disk Management | Partitions, volumes, drive letters, expand / shrink partitions. |
| Windows Server Backup | (Server-only) built-in backup tool. |

**3. Services and Applications:**

| Tool | What it does |
| --- | --- |
| Services | The service control panel. Each service has: display name, service name, path to executable, startup type (Automatic / Manual / Disabled), status. |
| WMI Control | Windows Management Instrumentation — scripting interface for managing Windows. Used by PowerShell, historically by `wmic`. |

**System Information (`msinfo32.exe`).** Read-only dump of everything about the system: OS version, hardware summary, installed hotfixes, running services, driver details, network adapter config. First stop when you land on a box you've never seen.

**Resource Monitor (`resmon.exe`).** Deeper than Task Manager. Shows exactly which processes are hitting CPU / disk / network / memory, which files they have open, which TCP/UDP ports they're listening on. Essential for spotting suspicious network activity or unknown processes reading sensitive files.

**Command Prompt (`cmd.exe`).** Windows' shell. Not as capable as PowerShell but still ubiquitous. Every command has a `/?` help flag.

| Command | Use |
| --- | --- |
| `hostname` | Show computer name |
| `whoami` | Show current user |
| `whoami /groups` | Show group memberships |
| `ipconfig` | Network config — IPv4, gateway, subnet |
| `ipconfig /all` | Full network detail — MAC, DNS, DHCP, lease |
| `ipconfig /?` | Help |
| `netstat` | Active network connections |
| `netstat -a` | All listening + established sockets |
| `netstat -b` | Show which executable owns each connection |
| `net user` | List local users |
| `net user <name>` | Detail on one user |
| `net localgroup` | List local groups |
| `net localgroup Administrators` | List Administrators-group members |
| `net share` | List shared folders |
| `net help <cmd>` | Help for a net sub-command (not `/?`) |
| `cls` | Clear the screen |

**Windows Registry.** A hierarchical database that stores config for the OS, drivers, applications, and users. Windows reads it constantly. Five root keys (hives):

| Hive | What lives there |
| --- | --- |
| `HKEY_LOCAL_MACHINE` (HKLM) | System-wide settings, hardware, installed software |
| `HKEY_CURRENT_USER` (HKCU) | Settings for the currently logged-in user |
| `HKEY_USERS` (HKU) | All loaded user profiles |
| `HKEY_CLASSES_ROOT` (HKCR) | File associations, COM objects |
| `HKEY_CURRENT_CONFIG` (HKCC) | Current hardware profile |

**Registry Editor (`regedit.exe`).** GUI to view and edit the registry. Powerful and dangerous — wrong changes brick the machine.

## Mental model

WF1 taught you the surfaces; WF2 teaches you the admin tools that operate on those surfaces. Same pattern as the Linux Fundamentals series — Part 1 shows you the filesystem and users, Part 2 shows you the tools that manage them.

Everything in this room is a **launcher for a system component**. The Tools tab of msconfig even hands you the exact command lines for launching each admin utility. Once you know the executable names — `msconfig`, `compmgmt.msc`, `taskschd.msc`, `eventvwr.msc`, `lusrmgr.msc`, `perfmon`, `msinfo32`, `resmon`, `regedit`, `services.msc`, `devmgmt.msc`, `diskmgmt.msc` — Win+R (Run) takes you anywhere in the OS in two seconds.

The registry is the mental-model unlock: **almost every setting in every one of these GUIs is really just editing a registry key**. Understanding registry structure means understanding where Windows actually stores its state.

## Offensive angle

| Concept | Attacker use |
| --- | --- |
| `msconfig` → Services / Startup | Enumerate what starts at boot. Attackers with admin add themselves here for persistence. |
| Crash dumps | Kernel / complete memory dumps contain everything in memory at crash time — including credentials, session keys, decrypted data. Attackers with SYSTEM can trigger and exfiltrate a dump; forensic analysts do the opposite. |
| Environment variables | `echo %USERPROFILE%`, `echo %APPDATA%` — first commands after landing. Malware uses `%TEMP%` for staging. |
| Task Scheduler | Persistence: `schtasks /create /tn Updater /tr C:\Users\Public\rev.exe /sc onlogon /ru SYSTEM`. Runs at every login as SYSTEM. First place blue-team checks after suspected compromise. |
| Event Viewer | Attacker: `wevtutil cl Security` clears the Security log (leaves an Event ID 1102, but the previous events are gone). Defender: watch for that 1102, watch for 4624/4625 auth anomalies. |
| Shared Folders (`C$`, `ADMIN$`) | Administrative shares. `net use \\<host>\C$ /user:admin <password>` — lateral movement over SMB with any local admin credential. |
| `net user` / `net localgroup` | Post-compromise enumeration to identify other accounts and admin-group members. |
| `whoami /groups` and `whoami /priv` | Enumerate exactly what your current token can do. `SeImpersonatePrivilege` is a classic path to SYSTEM (Potato attacks). |
| `netstat -ano` / `netstat -b` | Find backdoor listeners and C2 sessions. `netstat -ano` shows PIDs; cross-reference with Task Manager to spot rogue processes. |
| Services misconfigurations | Weak service permissions (`accesschk.exe` from Sysinternals), unquoted service paths, DLL hijacking targets. Classic Windows privilege escalation surface. |
| WMI / `wmic` | Living-off-the-land — `wmic process call create "cmd.exe /c ..."` executes with SYSTEM if run in the right context. `wmic` is deprecated but still on most systems. |
| Resource Monitor → Network | Spot outbound C2 traffic: unknown process making persistent HTTPS connections to a suspicious IP. |
| Registry — Run keys | The single most common Windows persistence location: `HKLM\Software\Microsoft\Windows\CurrentVersion\Run` and `HKCU\...\Run`. Everything listed here runs at logon. Autoruns from Sysinternals surfaces all of them. |
| Registry — Services | `HKLM\SYSTEM\CurrentControlSet\Services\<name>` — services live here. Modify to hijack a legit service or add a new one. |
| Registry — credential caches | `HKLM\SECURITY`, `HKLM\SAM` — password hashes. Locked to SYSTEM but dumpable via `reg save` + Mimikatz / secretsdump.py. |
| Registry — AlwaysInstallElevated | `HKLM\Software\Policies\Microsoft\Windows\Installer\AlwaysInstallElevated` = 1 means MSI installers run as SYSTEM. Trivial privesc win if set. Always check. |
| `regedit` alternatives from cmd | `reg query`, `reg add`, `reg save`, `reg delete` — all doable from an unattended shell where GUI isn't available. |

The theme: every Windows tool in this room has a defensive use (see what's running, see what's shared, see who logged in) and an offensive counterpart (add persistence, harvest credentials, cover tracks). Learn both sides.

## Practical — fast enumeration on a fresh box

```cmd
:: who am I, what groups, what privileges
whoami
whoami /groups
whoami /priv

:: what other accounts exist
net user
net localgroup Administrators

:: network config and listening ports
ipconfig /all
netstat -ano

:: shares (default + custom)
net share

:: scheduled tasks (persistence check)
schtasks /query /fo LIST /v

:: services
sc query state= all

:: quick system fingerprint
systeminfo
msinfo32

:: registry Run keys — persistence hotspot
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

That's the first-minute enumeration loop on any Windows box, offensive or defensive.

## What clicked

The **msconfig → Tools tab** trick. It hands you the actual command lines for every admin tool on the system with descriptions. That's essentially a self-documenting cheat sheet for launching things via Run. Before this I was memorising `compmgmt.msc`, `eventvwr.msc`, `services.msc` in isolation — turns out Windows itself keeps the full list.

The **registry as the underlying truth** was the other click. GUI settings, group policies, service configurations, installed programs, user preferences — they're all just registry keys presented through different interfaces. `regedit` is the raw view; everything else is a wrapper.

## What to revisit

- **PowerShell equivalents** — `Get-Service`, `Get-ScheduledTask`, `Get-EventLog` / `Get-WinEvent`, `Get-ItemProperty` for the registry. GUI clicks don't scale; PowerShell does.
- **Sysinternals suite** — Autoruns (persistence audit), Process Explorer (Task Manager on steroids), TCPView (Resource Monitor's Network tab, standalone), procmon (event tracing). Not in the room but the actual tools sysadmins reach for.
- **Event Viewer specifics** — Event IDs 4624 (successful logon), 4625 (failed logon), 4672 (special privileges assigned), 1102 (log cleared). The alphabet of Windows blue-team work.
- **Registry hive files on disk** — `C:\Windows\System32\config\SAM`, `SYSTEM`, `SECURITY`. Where the hives actually live, and what you can do with them offline.
- **Group Policy** (`gpedit.msc`) — not in this room but the next layer up. GPOs push settings into the registry across a fleet.
