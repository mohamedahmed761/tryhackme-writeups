# Windows Fundamentals 1

**Path:** Cyber Security 101 → Windows and AD Fundamentals
**Room:** [Windows Fundamentals 1](https://tryhackme.com/room/windowsfundamentals1xbx)
**Difficulty:** Info · **Time:** ~30 min · **Completed:** 2026-07-21

> Related: [windows-basics.md](../pre-security/windows-basics.md) covers the same core Windows concepts from the newer Pre-Security path. See [windows-commands.md](../references/windows-commands.md) for the cheat sheet.

## What the room covered

A guided walk through the core surfaces of a Windows system: editions and versions, the desktop, the NTFS file system (with Alternate Data Streams), the `C:\Windows\System32` folder and `%windir%` environment variable, user accounts and groups via `lusrmgr.msc`, User Account Control (UAC), Settings vs Control Panel, and Task Manager.

## Key concepts

**Windows editions.** Home and Pro are the two main flavours for desktop. **BitLocker** (full-disk encryption) is a Pro-only feature. Windows Server (Server 2019 in the lab, 2025 current) uses the same kernel but strips desktop niceties and adds server roles.

**NTFS — the New Technology File System.** The default modern Windows filesystem. Older FAT16/FAT32 (still on USB sticks and SD cards) and HPFS came before it.

NTFS improvements over FAT:
- **Journaling** — keeps a log of pending changes, so failures can be repaired instead of corrupting the drive.
- **Files >4 GB.**
- **Per-file permissions.**
- **Folder / file compression and EFS encryption.**

**NTFS permissions** are richer than the Linux `rwx` model. Six standard permissions per file / folder:

| Permission | Meaning |
| --- | --- |
| Full control | Read, write, execute, delete, take ownership, change permissions |
| Modify | Read, write, execute, delete |
| Read & Execute | Read + run executables |
| List folder contents | See what's in a folder (folders only) |
| Read | Read contents / attributes |
| Write | Create / modify contents |

View them via right-click → **Properties** → **Security** tab.

**Alternate Data Streams (ADS).** An NTFS feature: every file has at least one data stream (`$DATA`), and NTFS lets a file carry additional streams that Windows Explorer doesn't show by default. Legitimate use: downloaded files get a `Zone.Identifier` stream tagging them as web-sourced. Illegitimate use: malware hides secondary payloads in ADS so they don't show up in a normal file listing. PowerShell can read them.

**The Windows folder.** `C:\Windows`, referenced by the environment variable `%windir%`. Environment variables let scripts and programs find system paths regardless of where Windows was installed.

**`C:\Windows\System32`.** Holds the critical OS binaries and DLLs. Deleting things from here breaks the OS. Almost every Windows administrative tool lives here — `cmd.exe`, `regedit.exe`, `taskmgr.exe`, `control.exe`, and so on.

**User accounts.**

| Account type | What they can do |
| --- | --- |
| Administrator | Add / delete users, change groups, change system settings, install software |
| Standard User | Change only their own files and personal settings, no system-wide changes |

Every user gets a profile under `C:\Users\<username>` with standard subfolders (Desktop, Documents, Downloads, Music, Pictures).

**`lusrmgr.msc` — Local Users and Groups.** Run via **Win+R** → `lusrmgr.msc`. Two folders: **Users** and **Groups**. Users get added to groups; groups have permissions; users inherit the sum of their group memberships. A single user can belong to multiple groups.

**UAC — User Account Control.** Even if you're logged in as an Administrator, your normal session runs with **standard privileges**. Actions that need elevation (installing software, modifying system files, changing settings) trigger a UAC prompt asking you to consent. This is why programs that require admin have the little shield icon on their launcher.

The built-in local Administrator account is exempt from UAC by default — it always runs elevated, which is why it's typically disabled and why compromising a normal admin-tier account isn't as immediately game-over as it sounds.

**Settings vs Control Panel.**
- **Settings** — the modern Windows 10 / 11 configuration hub.
- **Control Panel** — the legacy configuration tool. Still present because some settings haven't been ported yet. Launched via `control.exe` or `control`.

**Task Manager.** Live view of processes, performance, users, startup impact. Launch with **Ctrl+Shift+Esc** or right-click the taskbar. Six tabs: Processes, Performance, App history, Startup, Users, Details, Services.

## Mental model

Windows is layered like Linux, but the vocabulary is different:

| Concept | Linux | Windows |
| --- | --- | --- |
| Filesystem type | ext4 | NTFS |
| Superuser | root | Administrator + SYSTEM |
| Config store | text files under `/etc` | Registry (`HKLM`, `HKCU`), + files under `C:\ProgramData` |
| "Where the OS lives" | `/` | `C:\Windows` (`%windir%`) |
| "Where the tools live" | `/usr/bin`, `/sbin` | `C:\Windows\System32` |
| User homes | `/home/<user>` | `C:\Users\<user>` |
| Group management | `/etc/group`, `usermod -G` | `lusrmgr.msc` |
| Elevation | `sudo` | UAC prompt |

Once you build that mapping, everything Windows does has a Linux analogue and vice versa. Skills carry across; only the file paths and tool names change.

## Offensive angle

| Concept | Attacker use |
| --- | --- |
| Windows editions | Home editions lack BitLocker — easier to pull a disk and mount offline. Pro/Enterprise with BitLocker forces you to attack while the machine is running. |
| NTFS permissions | `icacls <path>` from cmd lists ACLs. Look for files writable by low-priv users but executed by services / other users. Classic Windows privesc lead. |
| Alternate Data Streams | Anti-forensics / persistence — `type payload.exe > legit.txt:hidden.exe` hides an executable inside another file. Doesn't show in Explorer. Run it via `wmic process call create` referencing the stream path. |
| `%windir%` and other env vars | `echo %USERPROFILE%`, `echo %APPDATA%` — fast enumeration once you have a shell. |
| System32 binaries | Living-off-the-land: `certutil.exe` for downloads, `bitsadmin.exe` for file transfers, `rundll32.exe` for code execution. LOLBAS project catalogues them. |
| `C:\Users\<user>` | User's Documents, Desktop, Downloads, plus `AppData` — loaded with credentials in browser data, session tokens, RDP files. |
| `C:\Users\Public` | World-writable. Classic staging directory for payloads. |
| `lusrmgr.msc` / `net user` | Post-compromise enumeration: who else has accounts, who's in Administrators, who's in Remote Desktop Users. |
| Standard User accounts | If you land as one, the goal is UAC bypass or full privilege escalation. Common: unquoted service paths, weak service permissions, AlwaysInstallElevated, DLL hijacking. |
| UAC bypasses | Known bypass techniques (fodhelper, computerdefaults, sdclt) let an admin-tier process elevate without a prompt. Blue-team defence: set UAC to "Always notify". |
| Task Manager | Attackers hide their process by naming it after a legit one (svchost.exe in a wrong path). Defenders check the actual image path, not just the name. |
| Guest account | Disabled by default in modern Windows, but if enabled it's an unauthenticated foothold. Always check with `net user`. |

## Practical — what the room had you do

```cmd
:: check Windows edition
Win + R → msinfo32.exe

:: find the file system and permissions
Right-click C: → Properties
Right-click any file → Properties → Security tab

:: environment variable for the Windows folder
echo %windir%

:: view users, groups, memberships
Win + R → lusrmgr.msc

:: quick net-command equivalents
net user
net user <username>
net localgroup Administrators

:: launch programs by name
control.exe             :: Control Panel
taskmgr.exe             :: Task Manager
cmd.exe                 :: Command Prompt
```

## What clicked

The Linux-to-Windows translation. Coming from the Linux Fundamentals rooms, everything Windows does has a familiar analogue — `root` ↔ Administrator, `/etc` ↔ Registry, `/usr/bin` ↔ System32, `/home` ↔ `C:\Users`, `sudo` ↔ UAC. Once you see the same architecture through different vocabulary, Windows stops feeling like a separate mental universe.

ADS was the other click — the fact that NTFS files can carry hidden secondary streams that Explorer doesn't show is genuinely non-obvious coming from Linux, and it opens up a whole class of offensive tricks you don't have on ext4.

## What to revisit

- **PowerShell equivalents** — GUI tools like `lusrmgr.msc` map to `Get-LocalUser` / `Get-LocalGroup`. Learn those before automation and privesc rooms.
- **NTFS special permissions and inheritance** — room touched the six basic permissions but real ACL abuse (BUILTIN\Users with Modify on a service executable, etc.) is where privesc happens.
- **Alternate Data Streams by hand** — `dir /r`, `Get-Item -Stream *`, actually creating and running one. Advent of Cyber 2 Day 21 was called out in the room.
- **Windows Registry structure** — briefly mentioned as "where config lives" but Windows Fundamentals 2 goes deeper. HKLM vs HKCU is the split to internalise.
