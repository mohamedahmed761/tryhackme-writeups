# Operating System Security Reference

Core concepts and principles from TryHackMe Pre-Security: Operating System Security. Use as quick recall before later rooms involving privilege escalation, hardening, and exploitation.

## Authentication vs authorisation

**Authentication** — proving who you are (login, password, biometric, token).

**Authorisation** — what you're allowed to do once authenticated (file access, system actions).

Two separate checks. Bypass one and the other still applies.

## Privilege levels

**Linux:**
- Root (UID 0) — full system control
- Standard users — limited to their own files and standard system operations
- Service accounts — limited to specific services

**Windows:**
- Administrator — full system control
- Standard user — limited
- SYSTEM — highest privilege, used by OS processes (often higher than Administrator)

Offensive angle: attackers escalate from standard user → root/Administrator → SYSTEM.

## File permissions

**Linux — rwx model:**
- Three triplets: owner, group, other
- Each triplet: read (4), write (2), execute (1)
- Common examples:
  - `755` = owner rwx, group r-x, other r-x (typical executable)
  - `644` = owner rw-, group r--, other r-- (typical document)
  - `777` = world-writable (dangerous, attack target)

**Windows — ACL model:**
- More granular than Linux rwx
- Each file has Access Control List with specific permissions per user/group
- Permissions: Read, Write, Read & Execute, Modify, Full Control

## Password storage

**Linux:**
- `/etc/passwd` — user accounts (readable by all)
- `/etc/shadow` — password hashes (root only)
- Modern systems use bcrypt, SHA-512, Argon2

**Windows:**
- SAM file — local account hashes
- NTDS.dit — domain account hashes (on domain controllers)
- Historically used NTLM, now Kerberos for domain auth

Offensive angle: dumping these files is a common post-exploitation goal.

## Process isolation

Each process runs in its own memory space. One process can't read another's memory by default. Enforced by the kernel via memory management hardware (MMU).

Offensive angle: attackers look for ways to break this isolation (memory corruption bugs, shared resources, side channels).

## Common OS-level defences

1. **Permission enforcement** — file ACLs, process privileges
2. **Address Space Layout Randomisation (ASLR)** — randomises memory addresses to make exploits harder
3. **Data Execution Prevention (DEP)** — marks memory regions as non-executable
4. **Stack canaries** — values placed on the stack to detect buffer overflow attacks
5. **Mandatory Access Control (SELinux, AppArmor)** — additional permission layer on top of standard Linux permissions

## Common OS-level attack patterns

1. **Privilege escalation** — moving from low-privilege user to high-privilege (root/SYSTEM)
2. **Credential dumping** — extracting password hashes from /etc/shadow, SAM, memory
3. **Kernel exploitation** — exploiting bugs in the kernel itself to bypass all user-space security
4. **Misconfigurations** — world-writable files, weak permissions, default credentials
5. **Persistence** — installing backdoors that survive reboots (startup scripts, scheduled tasks, services)

## Key file paths to remember

**Linux:**
- `/etc/passwd` — user accounts
- `/etc/shadow` — password hashes
- `/etc/sudoers` — sudo configuration
- `/etc/group` — group membership
- `/root/.ssh/` — root's SSH keys (high-value target)

**Windows:**
- `C:\Windows\System32\config\SAM` — local password hashes
- `C:\Windows\NTDS\NTDS.dit` — domain hashes (on DCs)
- `C:\Windows\System32\drivers\etc\hosts` — local DNS overrides

## What to use this reference for

Quick recall before:
- Linux PrivEsc room (Jr Pentester)
- Windows PrivEsc room (Jr Pentester)
- Active Directory rooms
- Any OSCP-style exam prep
- Interviews where OS security questions come up
