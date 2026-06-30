# Become a Hacker

**Path:** Pre-Security → Attacks and Defenses
**Room:** [Become a Hacker](https://tryhackme.com/room/becomeahacker)
**Difficulty:** Easy · **Time:** ~30 min · **Completed:** 2026-06-15

## What the room covered

What offensive security actually is, the core vocabulary (red team, pentest, vulnerability, exploit, scope), the hacker mindset (assume nothing, test the unexpected, chain weaknesses), and a hands-on web app exercise: find a hidden `/login` page with Gobuster, then brute-force the `admin` password with Hydra to demonstrate chained exploitation.

## Key concepts

**Core terms.**

| Term | Meaning |
| --- | --- |
| **Red Teaming** | Authorised simulation of a real adversary against the full defensive stack. Bigger scope, more realistic, often covert from the blue team. |
| **Penetration Test** | Authorised, structured assessment to find and exploit vulnerabilities in a defined scope. Usually shorter and more focused than red teaming. |
| **Vulnerability** | A weakness or flaw in a system, app, or configuration that could be abused. |
| **Exploit** | A technique that takes advantage of a vulnerability to achieve an outcome (access, data, control). |
| **Scope** | The contractual line for what's in-bounds and what's off-limits during an engagement. Cross it and you're not a pentester anymore — you're a criminal. |

The one rule that runs through all of these: **permission**. Same techniques as a malicious attacker, with a signed contract authorising them.

**The hacker mindset.**
- **Ask questions.** Don't assume a feature works as designed — ask what happens if it doesn't.
- **Test the unexpected.** Try inputs the developers never imagined.
- **Chain small weaknesses.** A hidden login page is harmless. A hidden login page + a weak password list = full admin access.
- **Think like the adversary.** How would a real attacker approach this?

**The domino model.** Single vulnerabilities are often low-impact. Chained vulnerabilities cascade. Modern attacks almost always combine several small issues — a leaked username here, a weak password there, a misconfigured permission letting you pivot. The hacker's value isn't finding one bug; it's seeing how the bugs line up.

## Mental model — the methodology

The room walks through the two phases that every offensive engagement starts with:

1. **Reconnaissance / discovery** — what exists, what's exposed, what's reachable. The room used directory enumeration: manually trying URLs (`/sitemap`, `/login`, `/admin`) and then automating with Gobuster against a wordlist.
2. **Exploitation** — turn the discovery into access. The room used a dictionary attack with Hydra against the discovered login page to find valid credentials.

Real engagements add more phases (scoping, weaponisation, post-exploitation, reporting), but the discovery → exploitation rhythm sits at the core of all of them. Once you see it, every CTF and pentest reads the same way.

**Manual vs automated.** Manual is fine for short lists — five URLs, five passwords. The moment you need to test hundreds or thousands, you reach for a tool. Gobuster for directories. Hydra (or similar) for credentials. Every offensive specialty has its automated workhorses.

## Offensive angle

| Concept | Real-world attacker use |
| --- | --- |
| Hidden / forgotten endpoints | Old admin panels, debug pages, staging environments, `/.git/`, `/backup.zip`. Companies leave them up and forget. Gobuster / ffuf / dirsearch find them. |
| Directory enumeration wordlists | SecLists' `common.txt` / `raft-large-directories.txt` / `dirbuster` lists are the same ones real attackers use. Same tools, same wordlists, different intent. |
| Dictionary attack | Hydra against SSH / RDP / web logins, Hashcat / John against captured hashes. Defenders' best counter is rate limiting + account lockout + MFA. |
| Credential reuse | Password from one breach becomes the foothold on a different service. Hydra / Burp Intruder spray a leaked password across thousands of usernames. |
| Chained weaknesses | The kill chain: recon → initial access → privesc → lateral movement → objective. Each link is a separate weakness; the chain is the impact. |
| "Think like an adversary" | Threat modelling. Before defenders can defend, they have to imagine the attack paths. Same mental work, opposite goal. |
| Permission / scope | The legal line between pentester and criminal. "I was just testing" is not a defence without a signed contract. Always operate inside scope. |

## Practical — the onlineshop.thm walkthrough

The room's hands-on chained two weaknesses to demonstrate the kill chain in miniature.

**Step 1 — find a hidden login page.**

```bash
# manual
http://www.onlineshop.thm/sitemap   # 404
http://www.onlineshop.thm/admin     # 404
http://www.onlineshop.thm/login     # 200 — hit!

# automated
gobuster dir --url http://www.onlineshop.thm/   -w /usr/share/wordlists/dirbuster/directory-list.txt
```

Gobuster walks the wordlist and reports any URLs that respond with non-404 status codes. The status code (200 = OK, 301 = redirect, 403 = forbidden, 404 = missing) tells you whether the page exists and how it's configured.

**Step 2 — brute-force the login.**

```bash
# manual: try a handful by hand
# admin / abc123      → fail
# admin / 123456      → fail
# admin / password    → success

# automated: full dictionary attack
hydra -l admin -P passlist.txt www.onlineshop.thm http-post-form   "/login:username=^USER^&password=^PASS^:F=incorrect" -V
```

Hydra submits each password from the wordlist to the login form and checks the response. The `F=incorrect` flag tells Hydra what a failed login looks like — anything different is treated as success.

Two small issues (an exposed login + a weak password) chain into full account access. That's the kill chain in three lines of bash.

## What clicked

The domino framing. Before this, "hacker" felt like one big single moment of cleverness. Seeing it broken into "find something exposed" → "find something weak" → "chain them" made it look like a process I could actually learn and repeat instead of a talent I either had or didn't. Real attackers aren't smarter; they're more methodical.

## What to revisit

- Gobuster syntax and wordlist choice — SecLists is huge and picking the right list for the target matters.
- Hydra's protocol modules — the room used `http-post-form`, but Hydra also does SSH, FTP, SMB, etc., and each has its own quirks.
- Status codes properly (`200` / `301` / `401` / `403` / `404` / `500`) — reading the response codes correctly is half of enumeration.
- The legal line. Pentesting without explicit, written scope is illegal. Always confirm the engagement letter before touching anything.
