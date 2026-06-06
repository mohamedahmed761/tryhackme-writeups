# Operating System Security

**Path:** Pre-Security → Operating Systems Basics
**Room:** [Operating System Security](https://tryhackme.com/room/operatingsystemsecurity)
**Difficulty:** Easy · **Time:** ~60 min · **Completed:** 2026-06-06

## What the room covered

What an OS is, the CIA triad (Confidentiality, Integrity, Availability), the three classic OS-level weakness areas (weak authentication, weak file permissions, malicious programs), and a hands-on SSH lab where you brute-force a weak password, find leaked credentials in command history, and escalate to root.

Tasks: intro to OS security, common examples (auth / permissions / malware), practical (SSH into a Linux box and pivot to root).

## Key concepts

**CIA triad — what security is protecting.**
- **Confidentiality** — private data only reachable by intended people.
- **Integrity** — files can't be tampered with in storage or in transit.
- **Availability** — the system is usable when you need it.

Every attack maps to breaking one of these three.

**Authentication factors.**
- *Something you know* — password, PIN.
- *Something you are* — fingerprint, face.
- *Something you have* — phone, token, smart card.

Passwords are the most common, so they're the most attacked. NCSC's top 20 leaked passwords are all trivial (`123456`, `password`, `qwerty`, `iloveyou`, `dragon`, `monkey`). Even "complex-looking" ones like `qwertyuiop` and `1q2w3e4r5t` are keyboard walks — predictable.

**Principle of least privilege.** Files should only be accessible to people who actually need them for their job. Weak file permissions break confidentiality (attacker reads what they shouldn't) and integrity (attacker writes what they shouldn't).

**Malicious programs map onto CIA.**
- Trojans → confidentiality / integrity (attacker can read and modify your files).
- Ransomware → availability (encrypts files, demands payment for the key).

**Privileged accounts.**
- Linux / Android / Apple: **root** — unrestricted.
- Windows: **Administrator** — unrestricted (and **SYSTEM** is even higher, used by OS processes).

The goal of post-exploitation is almost always reaching one of these.

## Mental model — how the OS enforces security

The OS is the gatekeeper between hardware and apps. Security is layered — each layer slows the attacker down; none is enough alone.

1. **Authentication** — proves who you are at the door (password, key, biometric). If this layer fails, nothing else matters.
2. **Authorisation / file permissions** — once inside, the OS decides what each user can read, write, or execute. Linux uses the `rwx` model per owner/group/other; Windows uses ACLs that list per-user/per-group permissions.
3. **Process isolation** — every process runs in its own memory space, enforced by the CPU's memory management hardware. A misbehaving app can't read another app's memory by default.
4. **Privilege separation** — kernel space vs user space. Apps make system calls to the kernel rather than touching hardware directly. One crashed app doesn't take the whole machine down.
5. **Defence-in-depth additions** — ASLR randomises memory layout so exploits can't guess addresses, DEP marks memory as non-executable, stack canaries detect buffer-overflow attempts, and MAC frameworks (SELinux, AppArmor) add a second permission layer on top of standard Linux file permissions.

The attacker has to defeat each layer in sequence. Strong OS security means each is tightened, so even a successful breach at one level doesn't immediately yield root.

## Offensive angle

For each concept in the room, what the attacker is actually doing.

| Concept | Attacker move |
| --- | --- |
| Password authentication | Guess from top-N common-password lists; spray usernames against weak SSH; reuse passwords leaked from other breaches. |
| Username discovery | Read `/etc/passwd`; `whoami` once inside; OSINT on staff; literal sticky notes on monitors. |
| Weak file permissions | Hunt world-writable files, SUID binaries, weak ACLs, readable `/etc/shadow` copies. Each is a privilege-escalation lead. |
| Command history | After landing on a box, read `~/.bash_history` and run `history` — users often mistype passwords as commands. Free credentials. |
| Privileged accounts | Goal is always escalation: standard user → root / Administrator → SYSTEM. |
| SSH | Brute-force / dictionary against weak passwords; once in, pivot laterally to other accounts via `su -` using the same dictionary. |
| Ransomware | Attacker doesn't need files exfiltrated — destroying availability via encryption is enough to get paid. |

The pattern in this room — `ssh sammie@... → whoami → ls → cat → history → su - root` — is the baby version of a real intrusion: initial access via weak credential, look around, discover more credentials, escalate.

## Practical — SSH lab walkthrough

The room dropped a sticky-note clue: `sammie` and `dragon`. Username and (suspected) password. Log in:

```bash
user@AttackBox# ssh sammie@<MACHINE_IP>
sammie@MACHINE_IP's password: dragon
...
sammie@beginner-os-security:~$ whoami
sammie
sammie@beginner-os-security:~$ ls
country.txt  draft.md  icon.png  password.txt  profile.jpg
sammie@beginner-os-security:~$ cat draft.md
# Operating System Security
Reusing passwords means your password for other sites becomes exposed if one service is hacked.
```

Then move laterally. Two other users — `johnny` and `linda` — are known to be sloppy. Try the top-N passwords against `johnny`:

```bash
# either externally
ssh johnny@<MACHINE_IP>
# or pivot from inside
su - johnny
```

Once in as `johnny`, run `history` — the room hints he mistyped the root password into the shell as a command. Read it straight from history. Then escalate:

```bash
su - root
cat /root/flag.txt
```

Three accounts, three layers of bad practice, full system in under five minutes.

## What clicked

The chained nature of it. Each "small" mistake compounds — a sticky note gives you a password, that gives you a foothold, the foothold gives you `history`, history gives you root. No single failure needed a 0-day or a sophisticated exploit. Real intrusions look like this much more than they look like Hollywood hacking.

## What to revisit

The defensive controls (ASLR, DEP, stack canaries, MAC) — the room mentions attack patterns but doesn't drill into the mitigations. Worth knowing before the Jr Pentester rooms so I understand what a hardened box looks like from the attacker side. Also: NTLM vs Kerberos on Windows, and where SAM / NTDS.dit sit — touched in the reference doc, not the room.
