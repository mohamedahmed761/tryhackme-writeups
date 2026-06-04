# Windows CLI Basics

**Path:** Pre-Security → Operating Systems Basics
**Room:** [Windows CLI Basics](https://tryhackme.com/room/windowsclibasics)
**Difficulty:** Easy · **Time:** ~60 min · **Completed:** 2026-05-25

## Mission 1 — Find the brief

Find `task_brief.txt` somewhere under the user folder and read it.

```cmd
C:\Users\Administrator> cd
C:\Users\Administrator
```

`cd` with no arguments prints where you are.

```cmd
C:\Users\Administrator> dir
... 16 directories visible

C:\Users\Administrator> dir /a
... 12+ hidden folders show up
```

`dir` lists. `/a` includes hidden items (anything Windows hides by default).

Move around with `cd <folder>` and `cd ..` to go up one level.

```cmd
C:\Users\Administrator> dir /s task_brief.txt
... full path printed
```

`dir /s` searches recursively from the current directory — Windows walks every subfolder.

```cmd
C:\Users\Administrator> cd <path_to_file>
C:\...> type task_brief.txt
... file contents printed
```

`type` prints a file's contents (Windows equivalent of Linux `cat`).

## Mission 2 — System report

Standard "what is this machine?" questions every analyst asks first.

```cmd
C:\Users\Administrator> whoami
<user>

C:\Users\Administrator> hostname
<computer-name>
```

```cmd
C:\Users\Administrator> systeminfo
... OS Name, OS Version, System Type (32-bit / 64-bit), and a lot more
```

`systeminfo` dumps everything; focus on **OS Name**, **OS Version**, **System Type**.

```cmd
C:\Users\Administrator> ipconfig
... IPv4 Address, Default Gateway, etc.
```

Look for **IPv4 Address** (the machine's IP) and **Default Gateway** (the router it talks through).

## Commands learned

| Command | Job |
| --- | --- |
| `cd` | Where am I (no args) |
| `cd <dir>` / `cd ..` | Move in / move up |
| `dir` / `dir /a` | What's here / include hidden |
| `dir /s <file>` | Recursive search |
| `type <file>` | Print file contents |
| `whoami` | Current user |
| `hostname` | Computer name |
| `systeminfo` | OS name, version, architecture |
| `ipconfig` | Network config (IPv4, gateway) |

## What clicked

Coming straight from Linux CLI Basics, the mapping was obvious — `dir` ≈ `ls`, `type` ≈ `cat`, `cd` works the same. Once that landed, `dir /s` felt like the Windows version of `find ~ -name ...` from the Linux room.

## What to revisit

`systeminfo` output — too many fields scrolling past. I want to skim it and pick out the three the room called out (OS Name / OS Version / System Type) without scrolling back up. Also worth getting comfortable with `ipconfig /all` for the fuller network view.
