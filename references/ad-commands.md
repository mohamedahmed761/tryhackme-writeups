# Active Directory commands

Personal cheat sheet for AD administration and enumeration. Grows as I work through more rooms.

## MMC snap-ins (Run → Win+R)

| Command | Opens |
| --- | --- |
| `dsa.msc` | Active Directory Users and Computers (ADUC) |
| `dssite.msc` | Active Directory Sites and Services |
| `domain.msc` | Active Directory Domains and Trusts |
| `adsiedit.msc` | ADSI Edit — raw LDAP editor for AD |
| `gpmc.msc` | Group Policy Management Console |
| `gpedit.msc` | Local Group Policy Editor |
| `certsrv.msc` | Certification Authority (ADCS) |
| `dnsmgmt.msc` | DNS Manager |
| `dhcpmgmt.msc` | DHCP Manager |

## `net` commands (domain-aware)

| Command | Use |
| --- | --- |
| `net user /domain` | List all domain users |
| `net user <name> /domain` | Detail on one domain user |
| `net group /domain` | List all domain groups |
| `net group "Domain Admins" /domain` | Members of Domain Admins |
| `net group "Enterprise Admins" /domain` | Members of Enterprise Admins |
| `net localgroup Administrators /domain` | Local admins on the current DC |
| `net accounts /domain` | Password policy for the domain |
| `net view /domain` | List domains visible on the network |
| `net view /domain:<name>` | List computers in a domain |
| `nltest /dclist:<domain>` | List domain controllers |
| `nltest /domain_trusts` | List trust relationships |

## PowerShell AD module

Requires `RSAT-AD-PowerShell`. Present on every DC.

| Command | Use |
| --- | --- |
| `Import-Module ActiveDirectory` | Load the module |
| `Get-ADDomain` | Info about the current domain |
| `Get-ADForest` | Forest info + list of domains |
| `Get-ADDomainController -Filter *` | List DCs |
| `Get-ADUser -Filter *` | Every user in the domain |
| `Get-ADUser -Identity <n> -Properties *` | Full detail on one user |
| `Get-ADGroup -Filter *` | Every group |
| `Get-ADGroupMember "Domain Admins"` | Members of a group |
| `Get-ADComputer -Filter *` | Every computer object |
| `Get-ADOrganizationalUnit -Filter *` | Every OU |
| `Get-ADTrust -Filter *` | Trust relationships |
| `Get-GPO -All` | List every GPO |

**Modifying (needs write permissions):**

| Command | Use |
| --- | --- |
| `New-ADUser` | Create a user |
| `Set-ADUser` | Modify a user attribute |
| `Add-ADGroupMember "Domain Admins" <n>` | Add user to group |
| `Set-ADAccountPassword` | Change password |
| `Unlock-ADAccount` | Unlock a locked account |
| `Enable-ADAccount` / `Disable-ADAccount` | Toggle account state |

## Group Policy

| Command | Use |
| --- | --- |
| `gpupdate` | Refresh Group Policy (only if changed) |
| `gpupdate /force` | Force full re-apply |
| `gpresult /r` | Resultant policy summary for current user |
| `gpresult /h report.html` | Full HTML report |

## Kerberos

| Command | Use |
| --- | --- |
| `klist` | List cached Kerberos tickets |
| `klist purge` | Clear cached tickets |
| `klist tgt` | Show the current TGT |
| `setspn -L <n>` | List SPNs registered to a user (kerberoast targets) |
| `setspn -Q */*` | Find every SPN in the domain |

## Well-known SIDs

| SID | Meaning |
| --- | --- |
| `S-1-5-18` | LOCAL SYSTEM |
| `S-1-5-19` | LOCAL SERVICE |
| `S-1-5-20` | NETWORK SERVICE |
| `S-1-5-32-544` | BUILTIN\Administrators |
| `S-1-5-21-...-500` | Domain Administrator |
| `S-1-5-21-...-502` | krbtgt |
| `S-1-5-21-...-512` | Domain Admins group |
| `S-1-5-21-...-519` | Enterprise Admins |

## Fast domain enumeration

```powershell
# Basics
whoami /all
systeminfo
nltest /dclist:$env:USERDOMAIN

# Users & groups
net user /domain
net group "Domain Admins" /domain
net group "Enterprise Admins" /domain

# Kerberoastable accounts (SPNs)
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

# AS-REP roastable accounts (no pre-auth)
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} -Properties DoesNotRequirePreAuth

# Trusts
Get-ADTrust -Filter *
nltest /domain_trusts /all_trusts

# Password policy
Get-ADDefaultDomainPasswordPolicy
net accounts /domain

# LAPS-managed passwords (if reader)
Get-ADComputer -Filter * -Properties ms-Mcs-AdmPwd
```

## Offensive tooling (future rooms)

| Tool | Purpose |
| --- | --- |
| **BloodHound / SharpHound** | Graph AD attack paths visually |
| **PowerView** | Offensive AD PowerShell module |
| **Mimikatz** | Credential dumping, Golden/Silver tickets, DCSync |
| **Rubeus** | Kerberoasting, AS-REP roasting, ticket forging |
| **CrackMapExec / NetExec** | Bulk cred spraying, SMB/WinRM enum |
| **impacket** | Python swiss army knife (GetNPUsers, secretsdump, psexec) |
| **certipy** | ADCS abuse (ESC1-ESC15) |
| **kerbrute** | User enum via Kerberos pre-auth |
| **evil-winrm** | Interactive WinRM shell |

## Sources

- TryHackMe — Active Directory Basics
- Microsoft Docs — Active Directory PowerShell module
- ADSecurity.org — Sean Metcalf's AD reference
- HackTricks — Active Directory Methodology
