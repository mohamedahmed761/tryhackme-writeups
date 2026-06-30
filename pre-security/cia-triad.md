# The CIA Triad

**Path:** Pre-Security → Attacks and Defenses
**Room:** [The CIA Triad](https://tryhackme.com/room/theciatriad)
**Difficulty:** Easy · **Time:** ~30 min · **Completed:** 2026-06-15

> See [operating-system-security.md](./operating-system-security.md) for deeper treatment of how each pillar is enforced at the OS level.

## What the room covered

The three pillars cyber security is built on — Confidentiality, Integrity, Availability — with examples of each from both physical and digital worlds, plus a drag-and-drop exercise mapping nine real incidents to the pillar they break.

## Key concepts

**Confidentiality** — only authorised people can read the data. Broken by eavesdropping, credential theft, leaks. Enforced by encryption and access controls.

**Integrity** — data isn't modified without permission. Broken by tampering (a grade changed after marking, a bank-transfer destination swapped in transit). Enforced by hashing, digital signatures, write-once storage.

**Availability** — the data and services are reachable when needed. Broken by outages, DoS / DDoS, ransomware. Enforced by redundancy, backup power, rate limiting, capacity planning.

All three matter at once. A bank can have perfect confidentiality and integrity, but if the doors are locked when you need cash, the service still failed you.

## Mental model — the CIA as an incident-response checklist

The room's key insight: CIA isn't just a definition, it's a mindset. When an incident hits, security professionals work down the same three questions:

1. Was sensitive data exposed to people who shouldn't see it? → **confidentiality** breach
2. Was data changed without authorisation? → **integrity** breach
3. Were users locked out of systems they need? → **availability** breach

The answers shape the response. A confidentiality breach triggers credential rotation and breach notifications. An integrity breach triggers data validation and rollback. An availability breach triggers traffic rerouting and capacity additions. Same framework, different playbooks.

Every security control you'll meet later — ACLs, firewalls, backups, MFA, hashing — is solving one or more of these three.

## Offensive angle

The other side of the triad: every attack is an attack on at least one of these pillars. Attackers think in CIA too — they just want to break it instead of preserve it.

| Pillar | Attacker goal | Typical attacks |
| --- | --- | --- |
| Confidentiality | Read what they shouldn't | Credential theft, sniffing on public Wi-Fi, SQL injection to dump tables, phishing, dumpster diving, shoulder surfing |
| Integrity | Change what they shouldn't | Man-in-the-middle to alter a transfer, modifying log files to cover tracks, defacing a website, tampering with database records |
| Availability | Make services unreachable | DoS / DDoS, ransomware encrypting files, deleting backups, physical sabotage (cutting cables, pulling plugs) |

Many real attacks hit more than one pillar at once. Ransomware breaks availability (your files are unreadable) and often confidentiality too (modern variants exfiltrate first, then encrypt — "double extortion"). A SQL injection attack might pull credit cards (confidentiality), modify account balances (integrity), and drop a table for fun (availability), all in the same session.

## What clicked

CIA gives you a vocabulary for what's actually at stake in any incident. "We got hacked" is useless. "Confidentiality was compromised — 50k customer records exfiltrated" is actionable. The triad lets you classify and prioritise without needing the technical detail of how the attack worked.

## What to revisit

Mapping specific attack types to multiple pillars at once — ransomware, supply-chain attacks, insider threats often hit two or three simultaneously. The room treats each pillar in isolation; real incidents rarely respect those boundaries.
