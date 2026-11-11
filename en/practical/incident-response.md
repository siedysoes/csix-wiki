# Incident Response Basics

Incident response is the practice of detecting, containing, eradicating, and recovering from security incidents in a controlled, documented way. A defined process matters because the first hours are chaotic — without one, teams destroy evidence, miss notification deadlines, and make decisions nobody can explain afterwards.

## What counts as an incident

An **event** is any observable occurrence in a system or network. An **incident** is an event that harms, or threatens to harm, the confidentiality, integrity, or availability of information — and therefore needs coordinated handling beyond routine operations.

Incidents include:

- Malware or ransomware on a workstation or server
- Confirmed unauthorized access, a successful phishing login, or leaked credentials
- Data exfiltration or unauthorized access to sensitive data
- Website defacement, denial of service, or an outage caused by an attack
- A public storage bucket exposing customer data

Not every event is an incident: a blocked phishing attempt, a single failed login, or a vulnerability patched in time are worth logging but not worth waking anyone up for. The test is whether the response requires people, decisions, and evidence — not whether the alert is noisy.

## The NIST SP 800-61 lifecycle

NIST SP 800-61 (Revision 2) describes a four-phase lifecycle; Revision 3, published in 2025, reworks the guidance around NIST CSF 2.0 while keeping the same practical activities. Treat the phases as a loop: findings from post-incident activity feed directly back into preparation.

| Phase | Goal | Typical activities |
|---|---|---|
| 1. Preparation | Be ready before anything happens | Playbooks, training, tabletop exercises, log and EDR/SIEM coverage, contacts and escalation paths, spare capacity |
| 2. Detection and analysis | Find the incident and understand its scope | Alert triage, log and packet review, timeline reconstruction, scoping affected hosts and data, severity rating |
| 3. Containment, eradication and recovery | Stop the spread, remove the cause, restore service | Isolation, blocking indicators, removing malware and persistence, patching the initial access vector, restoring known-good backups, watching for reoccurrence |
| 4. Post-incident activity | Learn and improve | After-action review, lessons learned, detection and playbook updates, metrics, evidence retention, legal and regulatory notifications |

The phases are not strictly sequential. Containment often starts while analysis is still running, and recovery can reveal new evidence that sends the team back to detection.

## The first hour

A practical checklist for the first 60 minutes after a report lands:

1. **Acknowledge and timestamp.** Record when the report arrived and who reported it, in UTC.
2. **Triage severity.** Is production down? Is data exposed? Is the attacker still active?
3. **Declare.** Assign an incident commander and open a single incident channel (bridge, chat room, ticket).
4. **Preserve evidence first.** Capture volatile data — memory, then network state — before touching disks.
5. **Contain without destroying evidence.** Isolate affected hosts at the network level (dedicated VLAN, switch port, firewall rule) rather than shutting them down, unless the host is actively encrypting data.
6. **Scope.** Identify affected systems, accounts, and data; search logs for the same indicators elsewhere.
7. **Notify.** Escalate to management, legal, and the privacy officer when personal data may be involved, and start any regulatory clocks.
8. **Document continuously.** The scribe maintains the timeline: action, actor, time, result.

Do not coordinate over a possibly compromised system. If email or corporate chat sits on an affected host, switch to an out-of-band channel immediately.

## Incident roles

| Role | Responsibility | Notes |
|---|---|---|
| Incident commander | Owns decisions: declare, contain, escalate, close | Technical skill optional; decisiveness and clear communication required |
| Scribe | Maintains the authoritative timeline | One person, writing continuously — nobody else logs |
| Communications lead | Updates executives, staff, customers, and regulators | Speaks from the approved message, not personal opinion |
| Technical lead / analysts | Investigate, contain, eradicate, recover | Report to the commander, not directly to executives |
| Legal / privacy / HR | Advise on obligations and personnel matters | Engaged early when data or employees are involved |

In a small team one person may hold several roles — but name them explicitly in writing, because "who decides?" is the question that stalls response fastest.

## Evidence: order of volatility

Collect from most volatile to least, because each step can destroy the data above it.

| Order | Type | Examples | How to capture |
|---|---|---|---|
| 1 | Memory | Process list, open connections, malware resident in RAM | Memory dump (`winpmem`, `avml`, `LiME`) before any reboot |
| 2 | Network state | Active connections, ARP and DNS caches, live traffic | `netstat -ano`, `ss -tunap`, `arp -a`, a packet capture |
| 3 | Disk and logs | Files, registry keys, persistent logs, disk image | Forensic image with a write blocker or `dd`/FTK Imager, then hash it |

Rules that go with it: work on copies, never on originals; hash every artifact (SHA-256) as you collect it; record who collected what and when; never investigate directly on the evidence machine.

## Communication rules

- Facts only, with timestamps. Separate what you observed from what you infer, and label inferences as hypotheses.
- One source of truth: the incident channel and the scribe's timeline. No side decisions in private messages.
- Need to know: do not broadcast compromise details company-wide; share on a need-to-know basis.
- External statements come only from the communications lead, agreed with legal — customers, regulators, and press never hear it first from an engineer.
- Write for readers who arrive later: auditors, courts, and your successor.
- Assume the attacker can read internal chat and email, and plan channels accordingly.

## Tabletop exercises

A tabletop is a facilitated discussion of a realistic scenario — no exploitation, no live systems, just the team talking through decisions on paper.

1. Pick a scenario that matches your risk: ransomware on a file server, a stolen admin token, an exposed customer database.
2. Set the scene in one paragraph and deliver an initial inject (for example, a help-desk ticket reporting encrypted files).
3. Ask the team what they would do — who declares, what is captured first, who is notified, and by when.
4. Add an inject every 10-15 minutes to force trade-offs: the backup is 48 hours old, the CEO wants a press statement.
5. Time-box the decision points and note where the team stalls or disagrees.
6. Close with an after-action review: what was unclear, what changes in the playbook, who owns each fix.

Run one at least quarterly. A twenty-minute exercise that reveals you have no list of critical systems is worth more than an eight-hour exercise that flatters the team.

## Scenario: the first hour of a phishing incident

A finance user reports that after opening an email link, they typed their password into a look-alike page. The scribe opens the timeline:

```text
09:07 UTC  Report received from finance user (email forwarded to security)
09:10 UTC  Incident declared: IR-014; commander and scribe named, channel opened
09:12 UTC  Password reset and active sessions revoked for the user
09:15 UTC  Sign-in logs pulled: access from 203.0.113.44 at 09:03 UTC
09:18 UTC  Memory and network state captured from the user's workstation
09:25 UTC  Same source address searched across all accounts: one further match
09:35 UTC  Management and privacy officer notified; second account disabled
```

The reset happened before memory capture because the compromised asset was the credential, not the workstation, and the attacker was using it right now. Order of volatility tells you what to collect from a system you are about to touch — it never overrides stopping an active intrusion.

## See also

- [The CIA Triad](../fundamentals/cia-triad.md)
- [Threat Landscape](../fundamentals/threat-landscape.md)
- [Malware Types](../fundamentals/malware.md)
- [Network Security Basics](../fundamentals/network-basics.md)
- [Building a Home Lab](hands-on-lab.md)
