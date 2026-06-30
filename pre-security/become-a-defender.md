# Become a Defender

**Path:** Pre-Security → Attacks and Defenses
**Room:** [Become a Defender](https://tryhackme.com/room/becomeadefender)
**Difficulty:** Easy · **Time:** ~30 min · **Completed:** 2026-06-15

> Pair this with [become-a-hacker.md](./become-a-hacker.md) — same kill chain, opposite side of the table.

## What the room covered

What defensive security is, what defenders are actually responsible for protecting, the five-phase defender workflow (prevention → detection → mitigation → analysis → response), and a hands-on city-mapping exercise where you label infrastructure components and match the right protections to each.

## Key concepts

**Defensive security in one line:** identify what needs protecting, build visibility into it, prevent what you can, detect what you can't prevent, and respond fast when something gets through. Goal aligns directly with the CIA triad.

**The five defender activities.** Every blue team workflow maps to one of these.

| Activity | What it means | Example tooling |
| --- | --- | --- |
| **Prevention** | Stop the attack before it lands | Firewalls, antivirus, patching, MFA, hardening |
| **Detection** | Spot suspicious activity once it's inside | SIEM, IDS / IPS, EDR, log monitoring |
| **Mitigation** | Contain the damage during an incident | Block traffic, isolate hosts, disable accounts |
| **Analysis** | Investigate what happened, how, and what was hit | Forensics, log review, malware analysis |
| **Response and Improvement** | Recover services, harden against repeat | Restore from backup, write new detection rules, patch the gap |

**The city analogy.** Defender thinking maps cleanly onto a city:

| Infrastructure | City equivalent | Common protections |
| --- | --- | --- |
| Employee devices | Homes | Antivirus, EDR, regular updates, user training |
| Web server | Public buildings / shops | Allow only safe traffic, TLS, WAF |
| Mail server | Post office | Spam filtering, attachment scanning, DMARC / SPF / DKIM |
| Firewall | City gate | Inbound / outbound rules, block known-bad IPs |
| The wider internet | Anything outside the walls | Restrict inbound, monitor egress |

**The four defender principles.**
- **Threat anticipation.** Look at your systems and ask "what if?" — walk the realistic paths an attacker might take.
- **Attack awareness.** Real attacks follow recognisable stages. Study common chains (MITRE ATT&CK, the kill chain) so you spot them mid-flight.
- **Risk prioritisation.** Not every asset is equal. Identify the crown jewels (domain controller, customer database, source code) and weight defences accordingly.
- **Continuous adaptation.** Defence isn't a one-time setup. Threats evolve; controls must too.

## Mental model — defence is the same kill chain in reverse

The Hacker room framed offensive work as a chain: recon → initial access → escalation → lateral movement → objective. Defence is the same chain seen from the other side — each step the attacker takes is a chance for the defender to **prevent** (stop it happening), **detect** (notice it happened), or **respond** (limit how far it goes).

The example from the room: malicious email attachment lands on a workstation → attacker steals credentials → pivots to the mail server → reaches the database. That's four discrete steps, four defensive checkpoints:

1. **Email filter** could have caught the attachment.
2. **EDR / antivirus** on the workstation could have stopped execution.
3. **MFA** on the mail server could have stopped the credential reuse.
4. **Database access controls and monitoring** could have caught the lateral movement.

No single layer is enough — that's why defence is layered. Even one working control breaks the chain. This is "defence in depth": assume any single control will fail and stack enough independent ones that the attacker has to defeat all of them to win.

## Offensive angle — what attackers do to each defence

The defender side only makes sense if you also know how each layer gets attacked. Same controls, viewed from the other table.

| Defence | How attackers get around it |
| --- | --- |
| Firewall rules | Tunnel out over allowed protocols (HTTPS, DNS), use cloud services as C2 relays, hit poorly-segmented internal traffic |
| Antivirus / EDR | Obfuscation, packing, living-off-the-land binaries (powershell, certutil), bring-your-own-vulnerable-driver (BYOVD) |
| Patching | Target zero-days, exploit the patch gap (the window between disclosure and rollout), hit unpatched edge devices |
| Spam filters | Convincing pretext, hijacked legitimate accounts, attachments that don't match malicious signatures |
| Logging / SIEM | Disable logging on compromised hosts, clear event logs, time attacks around shift changes, generate noise to bury real alerts |
| MFA | Phish the second factor (Evilginx), SIM-swap the phone number, push-bomb the user into approval fatigue |
| Backups | Encrypt or delete backups before deploying ransomware (now standard practice in ransomware ops) |

The defender's job is to know these moves and build controls that survive at least the obvious ones. The attacker's job is to find the layer that wasn't hardened. Both sides are reading the same playbook — the question is which side practised harder.

## Practical — city mapping and defending

The room had two hands-on exercises:

1. **Mapping** — drag infrastructure pieces (homes, post office, shops, gate, beyond-the-walls) to their real-world equivalents (workstations, mail server, web server, firewall, internet).
2. **Defending** — match the right defensive control to each piece: antivirus on workstations, spam filter on the mail server, allow-listed traffic on the web server, firewall rules at the gate.

The lesson sits in the exercise: there's no universal protection. You apply the control that fits the threat that fits the asset. Antivirus on the firewall would be useless. A spam filter on a web server makes no sense. Defenders match tool to target.

## What clicked

The "city" framing as a map, not a metaphor. Once you can label every system in your environment, *visibility* stops being a vague buzzword and becomes literal: do you have eyes on the post office? on the gate? on the houses? If the answer is no, the attacker has already won there — you just don't know yet. Defence starts with the inventory, every time.

## What to revisit

- The MITRE ATT&CK framework — the room mentions "common attack chains and frameworks" without naming this one. It's the canonical taxonomy of attacker techniques mapped to defensive countermeasures.
- SIEM tooling (Splunk, ELK, Sentinel) — the room talks about logs and alerts without showing the platforms that aggregate them.
- Defence-in-depth case studies. Look up a few breach reports (Equifax, Target, SolarWinds) and trace which layers held and which failed. The chains are always the lesson.
