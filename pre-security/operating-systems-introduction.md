# Operating Systems: Introduction

**Path:** Pre-Security → Operating Systems Basics
**Room:** [Operating Systems: Introduction](https://tryhackme.com/room/operatingsystemsintroduction)
**Difficulty:** Easy · **Time:** ~45 min
**Completed:** 2026-05-08

## What the room covered

What an operating system actually is, the split between kernel space and user space, the five core OS duties, the difference between GUI and CLI interaction, and the major families of operating systems out there (Desktop, Server, Mobile, Embedded, Virtual/Cloud). Hands-on portion used an Ubuntu Mate target machine — opened System Monitor to read OS version and RAM, browsed the file system, found a flag in a note inside another user's home directory.

The four tasks: intro, The Invisible Manager (what an OS does), OS Interaction and Landscape (GUI vs CLI plus the OS family overview), and conclusion.

## Key concepts

**The airport analogy.** Hardware = runways, planes, radar. Applications = airlines and passengers. OS = air traffic control directing all of it.

**Kernel space vs user space.** Kernel space has unrestricted hardware access — it's the locked-down core. User space is where regular apps run, with no direct hardware access. When an app needs to read a file or play sound, it makes a system call asking the kernel to do it on its behalf. This separation is why one crashing app doesn't take down the whole machine.

**The five OS duties.**
- Process management — schedules CPU time across running programs so multitasking feels seamless.
- Memory management — allocates RAM, isolates apps from each other's memory, falls back to virtual memory when RAM runs low.
- File system management — directories, paths, permissions, metadata.
- User management — accounts, authentication, who-can-access-what.
- Device management — drivers and a hardware abstraction layer so apps don't need to know which printer or mouse you plugged in.

**OS as security foundation.** Before any antivirus runs, the OS is already enforcing authentication, permissions, process isolation, and protection of system files.

**GUI vs CLI.** GUI = clicking icons, like tapping a destination in maps. CLI = typing exact commands, like entering raw GPS coordinates — more precise and faster once you know the syntax.

**OS families.**
- Desktop — Windows, macOS, Linux distros (Ubuntu, Debian, Fedora).
- Server — Windows Server, Linux server distros (Ubuntu Server, Debian, CentOS, Red Hat), Unix (AIX, Solaris).
- Mobile — Android, iOS.
- Embedded — Embedded Linux (OpenWrt, Yocto), Real-Time OS (FreeRTOS, VxWorks, QNX).
- Virtual/Cloud — cloud Linux distros (Amazon Linux, Rocky), container-optimized OSes (Alpine, Bottlerocket, Flatcar).

**Why so many.** Different environments value different things — a laptop wants user-friendliness, a server wants uptime, a phone wants battery life, an embedded device wants a tiny footprint. No one OS fits all, so an ecosystem evolved.

**Hands-on (Ubuntu Mate target).** Opened About This Computer / System Monitor to read OS version and allocated RAM. Checked the File Systems tab for the type of /dev/root. Browsed the Home directory, counted user folders, navigated into Alex's Documents to find a flag in note.txt.

## What clicked

The GUI side. Clicking through Ubuntu Mate to find System Monitor, opening the File Systems tab, walking into another user's home directory and reading the flag out of a text file — that all felt natural because it's the kind of clicking-around-folders I already do. The OS doing process scheduling and memory allocation underneath while I was just clicking icons made the "invisible manager" framing land.

## What to revisit

The CLI. The room only contrasted it with the GUI as a concept (typing exact commands vs clicking) and said the actual command practice comes later in the module. I want to come back here once I've done the Linux and Windows CLI rooms — re-read this section knowing what `ls`, `cd`, `cat` actually do, so the GPS-coordinates analogy fully makes sense from experience, not just description.
