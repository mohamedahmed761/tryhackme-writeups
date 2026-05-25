# Windows Basics

**Path:** Pre-Security → Operating Systems Basics
**Room:** [Windows Basics](https://tryhackme.com/room/windowsbasics)
**Difficulty:** Easy · **Time:** ~45 min · **Completed:** 2026-05-25
**Machine:** Windows Server 2019, auto-login as Administrator

## Mission — first day at TryHatMe

Role-play a new employee. Log into the workstation, learn the desktop, manage files, install an app, run the security tools.

## Exploring the workspace

Account types:
- **Guest** — temporary, minimal permissions.
- **Standard** — daily use, no system-wide changes.
- **Administrator** — full control. You're auto-logged in as this.

Desktop + Taskbar pieces: desktop icons, Start menu, Search, Task View, pinned apps, network/audio, date/time, notifications.

**About your PC** (Settings > System > About) — device name, installed RAM, Windows edition and version.

**File Explorer** — hierarchical folders (Desktop, Documents, Downloads). Onboarding folder:

```
C:\Users\Administrator\Desktop\TryHatMe Onboarding
```

`Welcome.txt` inside holds the first flag.

## Configuring and securing

**Updating** — Windows Update (Settings) handles the OS; apps update via the Store, built-in updaters, or manual installers.

**Installing** — Microsoft Store (curated) or an `.exe` / `.msi` from a trusted vendor. Hands-on: ran the `TryHatMeWelcome` installer from the onboarding folder → flag on completion.

**Uninstalling** — Store, Add or remove programs, Control Panel, or the app's own uninstaller.

**Settings vs Control Panel** — Settings is the modern config hub; Control Panel is the legacy one, still needed for some tasks.

**Task Manager** — five tabs: Processes, Performance, Users, Details, Services. Users tab shows who's logged in.

**Windows Security** — four sections: Virus & threat protection, Firewall & network protection, App & browser control, Device security. Ran a custom scan on the onboarding folder → it flagged the EICAR test file (a harmless standard AV test file).

**Windows Defender Firewall** — profiles: Domain / Private / Public. Advanced settings list inbound, outbound, and connection rules.

## Tools learned

| Tool | Use |
| --- | --- |
| About your PC | Device name, RAM, Windows edition |
| File Explorer | Browse the folder tree |
| Settings vs Control Panel | Modern vs legacy config |
| Task Manager | Processes, performance, logged-in users |
| Windows Security | AV scans, firewall, device security |
| Windows Defender Firewall | Network rules per profile |

## What clicked

It's the everyday Windows GUI, so it all felt familiar — desktop, File Explorer, Task Manager, Windows Security. The value was seeing the last room's OS concepts (process and memory management, user accounts, security) exposed as real tools you click.

## What to revisit

Nothing in particular — straightforward room, no sticking points.
