# Active Directory Basics

**Path:** Cyber Security 101 → Windows and AD Fundamentals
**Room:** [Active Directory Basics](https://tryhackme.com/room/winadbasics)
**Difficulty:** Easy · **Time:** ~30 min · **Completed:** 2026-08-03

> Sequel to [windows-fundamentals-1.md](./windows-fundamentals-1.md) and [windows-fundamentals-2.md](./windows-fundamentals-2.md). See [ad-commands.md](../references/ad-commands.md) for the AD cheat sheet.

## What the room covered

What a Windows domain is, what Active Directory is, and how a domain admin uses it: users and groups, OUs (Organizational Units), computers, GPOs (Group Policies), the authentication protocols (NTLM and Kerberos), and how domains join into trees, forests, and trust relationships. Practical section runs inside a domain controller VM — you delete an OU, delegate control, and reset a user's password with ADUC.

## Key concepts

**Windows Domain.** A group of Windows machines that share a central database of users, groups, computers, and policies. Instead of managing accounts on each machine individually, everything is centralized on a **Domain Controller (DC)**. That central store is the Active Directory database.

Why domains exist:

| Without a domain | With a domain |
| --- | --- |
| Every machine has its own user list | Log in on any machine with one AD account |
| Every admin task done per-machine | Push policies to all machines at once |
| Passwords desynced everywhere | Password change propagates domain-wide |

**Active Directory (AD).** The Microsoft directory service. Runs on a DC. Its heart is the **AD DS (Active Directory Domain Services)** database — basically the phonebook of the network. Everything is an **object** with **attributes**.

**Core AD objects:**

| Object | Represents |
| --- | --- |
| User | A person (or a service account) |
| Group | Collection of users/computers for permission grants |
| Computer | A domain-joined machine |
| Contact | An external contact (no login) |
| Printer | A shared printer |
| Shared Folder | A shared network folder |
| Organizational Unit (OU) | A container to organize other objects |
| Domain / Tree / Forest | Structural containers (see below) |

**Organizational Units (OUs).** Folders inside AD used to group objects logically — usually by department (Sales, IT, Marketing) or geography (London, NY). OUs are the target for **GPOs** and for **delegated control**. By default OUs have "Protect from accidental deletion" turned on — you disable it in Advanced Features before you can delete.

**Users vs. Groups.** Two broad categories:

- **Users** — people or service accounts. `Administrator`, `Guest`, `krbtgt` (built-in Kerberos service account) live at the top of AD.
- **Groups** — **Security groups** grant permissions; **Distribution groups** are email lists. Nested groups are normal (Domain Admins is a member of local Administrators on every domain-joined box, for example).

**Default privileged groups to know:**

| Group | Power |
| --- | --- |
| `Domain Admins` | Full control over the domain — the crown jewels |
| `Enterprise Admins` | Full control over the entire forest |
| `Schema Admins` | Can modify the AD schema itself |
| `Account Operators` | Create/modify most user accounts |
| `Server Operators` | Administer domain controllers (no config changes) |
| `Backup Operators` | Read any file for backup — useful for AD dumps |
| `Print Operators` | Manage printers (can load drivers → code execution) |
| `DnsAdmins` | Manage DNS — historically a privesc path to DC via DLL |

**Computers.** Every domain-joined machine is also an AD object. `Domain Controllers` is its own OU. Computers auto-create their own passwords and rotate them every 30 days.

**Group Policy (GPO).** The mechanism for pushing configuration to users and computers. A GPO is a bundle of settings; you link it to an OU (or the domain root or a site), and every object under that link inherits the settings. Policies refresh roughly every 90 minutes, or on demand via `gpupdate /force`.

What you can push via GPO: password policy, firewall rules, mapped drives, login scripts, software installs, registry edits, USB blocking, screen lockout, `hosts` file, scheduled tasks. Basically anything Windows will read.

**Authentication protocols:**

| Protocol | How it works | Notes |
| --- | --- | --- |
| **Kerberos** | Ticket-based. User authenticates to KDC (part of DC) once, gets a **TGT** (Ticket Granting Ticket). To access a service they trade the TGT for a **TGS** (Ticket Granting Service ticket) for that specific service. No password crosses the wire after the initial auth. | Default since Windows 2000 |
| **NTLM** | Challenge-response with the user's password hash. DC verifies the response. Hash-based, not password-based on the wire. | Legacy, still enabled everywhere for backward compatibility |

**Trees, Forests, Trusts:**

| Structure | What it is |
| --- | --- |
| **Domain** | The base unit — one AD database with its own DNS namespace |
| **Tree** | Domains sharing a contiguous namespace (e.g. `acme.com`, `uk.acme.com`, `sales.uk.acme.com`) |
| **Forest** | One or more trees. The forest is the outer security boundary |
| **Trust** | Explicit trust link between domains — users in one can be authenticated by the other. One-way or two-way, transitive or non-transitive |

## Mental model

Think of AD as a login server + policy pusher + directory database rolled into one. Everything a Windows admin cares about — who exists, who can log in where, what config each machine runs — lives in AD. The DC is the machine that speaks the AD protocol.

**Compare it to what you already know:**

| Linux world | Windows/AD equivalent |
| --- | --- |
| `/etc/passwd`, `/etc/group` (per machine) | AD user/group objects (domain-wide) |
| LDAP + Kerberos (FreeIPA / OpenLDAP) | AD DS + Kerberos KDC (same idea, Microsoft build) |
| Ansible / Puppet pushing configs | GPO |
| `sudoers` on each host | AD group membership + local admin GPO |
| SSH key on every box | Kerberos TGT once, then tickets everywhere |

The Ansible analogy is the one that clicks: **GPO is Microsoft's config-as-policy engine**. Link a GPO to an OU and every machine below it complies. Break the GPO and you break login for 10,000 machines at once. This is why GPO abuse is a favourite persistence trick post-domain-compromise.

**The trust hierarchy** — domain → tree → forest — exists because big orgs merge, acquire, or split. Two companies that merge don't collapse their AD; they set up a trust and let users from Company B authenticate against Company A's DC. The forest is the true security boundary: compromise the forest root and you own everything.

## Offensive angle

AD is a huge attack surface. This intro room only scratches the surface, but the primitives introduced here are the ones offensive AD tooling operates on.

| Concept | Attacker use |
| --- | --- |
| User enumeration | `net user /domain`, PowerView `Get-DomainUser`, LDAP queries, BloodHound ingestor — first thing after landing in a domain. |
| Group enumeration | `net group "Domain Admins" /domain` — find who to target. `Domain Admins`, `Enterprise Admins`, `DnsAdmins`, `Account Operators` are all high-value. |
| Kerberos TGT theft | `Mimikatz sekurlsa::tickets /export` from an admin session — impersonate any authenticated user. |
| **Pass-the-Hash** | NTLM lets you authenticate with the raw hash — no need to crack it. Any dumped NTLM hash is a login. |
| **Pass-the-Ticket** | Import a stolen Kerberos ticket and use it directly. |
| **Kerberoasting** | Request TGS tickets for service accounts (SPNs), extract them, crack offline. Service accounts often have weak, unrotated passwords. |
| **AS-REP Roasting** | Users with "Do not require Kerberos preauthentication" leak crackable hashes when you request their AS-REP. `GetNPUsers.py`. |
| **Golden Ticket** | Forge a TGT for any user using the `krbtgt` account's NT hash — domain persistence for as long as krbtgt goes unrotated. |
| **Silver Ticket** | Forge a TGS for a specific service using that service account's hash. Quieter than golden. |
| **DCSync** | Impersonate a DC and pull hashes for any account via the replication protocol. Needs "Replicating Directory Changes" rights. |
| **DCShadow** | Register a rogue DC and push arbitrary changes into AD, bypassing most logging. |
| GPO abuse | Modify a GPO you have write to — push a scheduled task or a startup script to every machine linked under it. Massive blast radius. |
| OU delegation abuse | If you have delegated control over an OU (like Task 4's Phillip over Sales), you can reset passwords for every user under it. Escalation to that user's rights. |
| Trust abuse | Cross-domain / cross-forest attacks — SID history injection, foreign group membership, extra SIDs. |
| ACL abuse | AD objects have ACLs like files. `GenericAll`, `WriteDACL`, `WriteOwner`, `ForceChangePassword` on the wrong object gives you the keys. BloodHound visualises these. |
| Unconstrained delegation | A computer with unconstrained delegation caches TGTs from any user who authenticates to it — lure a DA to auth to your box, steal their TGT. |

The theme: **AD's convenience features are its attack surface**. Delegation, group nesting, GPOs, Kerberos tickets, replication — all designed to make sysadmins' lives easier, all abusable.

## Practical — what Task 4 walks through

Inside the domain controller VM the practical exercise runs through **Active Directory Users and Computers (`dsa.msc`, aka ADUC)**:

1. **Open ADUC** — Server Manager → Tools → Active Directory Users and Computers, or run `dsa.msc`.
2. **View Advanced Features** — View menu → Advanced Features. Without this, the Security tab on OUs is hidden and "Protect from accidental deletion" can't be unticked.
3. **Delete an extra OU** — you have a stale department OU (from an old org chart). Right-click the OU → Properties → Object tab → uncheck "Protect object from accidental deletion" → delete.
4. **Move users to the right OU** — drag-and-drop, or right-click → Move.
5. **Delegate control of the Sales OU to Phillip (IT Support)** — right-click the Sales OU → Delegate Control → add Phillip → pick "Reset user passwords and force password change at next logon". This is the standard helpdesk delegation — IT can reset Sales user passwords, but nothing else.
6. **Reset a password** — as the delegated user, right-click a user → Reset Password.

The organisational chart used: Daniel (GM) at top; Mark under Marketing; Sophie (Sales Director) and Thomas (Sales Rep) under Sales; Phillip (IT Support), Mary (Server Admin), Claire (Domain Admin) under IT.

**Commands used from the DC:**

```powershell
# Open the classic GUI consoles
dsa.msc         # Active Directory Users and Computers
dssite.msc      # Sites and Services
domain.msc      # Domains and Trusts
gpmc.msc        # Group Policy Management

# Force a policy refresh
gpupdate /force

# List every user/group/computer in the domain
Get-ADUser -Filter *
Get-ADGroup -Filter *
Get-ADComputer -Filter *

# Who's in Domain Admins?
Get-ADGroupMember "Domain Admins"

# Trust list
Get-ADTrust -Filter *
```

## What clicked

Seeing **AD as "LDAP + Kerberos + GPO glued together"** made the whole thing much less magical. Before this room I thought of AD as a Windows-specific black box; now it's just a directory service (LDAP) with an auth server (Kerberos KDC) and a config-push engine (GPO) all running on the same server. Everything the Microsoft docs call "AD DS" is basically "an LDAP directory with Windows-flavoured schema".

The **OU-as-policy-target model** also clicked. OUs aren't just folders — they're the unit GPOs apply to. That's why every AD design guide obsesses about OU structure: your OU tree determines which policies land where. Get the OU wrong and you push "disable USB storage" to your sysadmins' laptops.

And the **kill chain from delegated helpdesk rights to full compromise** was eye-opening. Task 4 innocently grants Phillip password-reset rights over Sales. Real environments do this constantly for helpdesk roles — and every one is a potential lateral movement path if that helpdesk account is compromised.

## What to revisit

- **Kerberos flow end to end** — AS-REQ → AS-REP → TGS-REQ → TGS-REP → AP-REQ. Need to draw this from memory before touching Kerberoasting / Golden Ticket properly.
- **PowerView / BloodHound / Rubeus** — offensive AD tooling. Not in this room; next module (Compromising Active Directory) covers them.
- **AD Hardening Room** — the flip side, called out in Task 9 as the next step. Password policies, LAPS, tiered admin model, protected users group.
- **ADCS** (Active Directory Certificate Services) — not covered here at all. ESC1-ESC8 vulns are one of the hottest AD attack surfaces right now.
- **The specific answers I entered on TryHackMe** — methodology is what stuck; the trivia answers can be re-derived by revisiting ADUC.
- **PowerShell AD module** vs **RSAT-AD-PowerShell** vs old **`net` commands** — three ways to do the same lookup; know when each is available on a locked-down box.

## Sources

- TryHackMe — [Active Directory Basics](https://tryhackme.com/room/winadbasics)
- Microsoft Docs — [AD DS Overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview)
- [SpecterOps BloodHound docs](https://bloodhound.specterops.io/) (referenced for offensive angle)
- [ATT&CK: T1078.002 Domain Accounts](https://attack.mitre.org/techniques/T1078/002/)
