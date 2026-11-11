# Threat Landscape

The threat landscape describes who attacks, why they attack, and how they get in. Understanding it lets you prioritize defences: the controls that stop a bored teenager are not the same ones that stop a funded, patient group stealing trade secrets.

## Threat actors

| Actor | Typical motivation | Skill and resources | Example behaviour |
|---|---|---|---|
| Script kiddie | Notoriety, curiosity, fun | Low skill; uses pre-made tools and tutorials | Runs a downloaded DDoS tool or defaces a website with a public exploit kit |
| Hacktivist | Ideology, protest, publicity | Low to moderate; works in groups, strong at propaganda | Defacing sites, leaking documents to make a political point |
| Cybercriminal | Money | Moderate to high; organized, uses phishing, ransomware, affiliates | Ransomware extortion, carding, business email compromise |
| Insider | Money, revenge, negligence | Varies; already has legitimate access | Copying customer lists, misconfiguring a server, clicking a link |
| Advanced Persistent Threat (APT) | Long-term espionage or sabotage | High; patient, custom tools, targets specific organizations | Months of quiet access, living-off-the-land techniques, targeted exfiltration |
| Nation-state | Geopolitical advantage, intelligence, sabotage | Very high; legal cover, huge resources, zero-day exploits | Stealing defence research, disrupting critical infrastructure |

Notes:

- **Insiders** are the hardest to detect because their activity looks legitimate. The category includes malicious actors and simply careless ones.
- **APT** is a label for behaviour (persistent, targeted, advanced), not a synonym for "government". Some criminal groups show APT-like traits.
- **Nation-state** operations sometimes blur into APT; the distinction is who stands behind the group.

## Motives

Four motives explain most attacks:

1. **Money** — ransomware, fraud, selling access, cryptomining, extortion. The dominant motive for criminal groups.
2. **Espionage** — stealing secrets: military, commercial, diplomatic, or research data. Dominant for nation-states and some APTs.
3. **Sabotage** — destroying or disrupting operations, from wiping disks to halting production lines. Sometimes a bargaining chip (pay us and the plants restart).
4. **Ideology** — hacktivism, protest, or personal grievance. Usually seeks publicity rather than direct payment.

Secondary motives include notoriety, revenge, blackmail, and simply causing chaos. Ransomware crews are financially motivated but often use sabotage-like impact (wipers that destroy data) as leverage.

## Attack vectors

An **attack vector** is the specific path or method an attacker uses to reach a target.

- **Phishing** — malicious email, link, or attachment that tricks a user into executing code or handing over credentials. Still the most common initial access method.
- **Drive-by download** — visiting a compromised or malicious site that silently exploits a browser or plugin vulnerability, sometimes without any click beyond loading the page.
- **Supply chain** — compromising a trusted dependency, update server, or vendor. Because the payload arrives signed or expected, it bypasses many filters. The SolarWinds Orion compromise (2020) injected a backdoor into software updates distributed to thousands of organizations.
- **Physical** — tailgating into offices, stealing laptops, plugging in rogue USB devices, or tampering with hardware.
- **Cloud misconfiguration** — publicly readable storage buckets, over-permissive IAM roles, unpatched internet-facing services. No exploitation skill required, just discovery.

## Attack surface vs attack vector

These two terms are often mixed up:

- **Attack surface** is everything an attacker *could* try: every exposed port, endpoint, web form, API, user, laptop, and third-party integration.
- **Attack vector** is the one path they *actually* use.

Example: an online shop exposes port 22 (SSH), 443 (HTTPS), a customer login form, an admin panel, a payment API, and a third-party chat widget. That entire list is the attack surface. An attacker who sends an email with a fake invoice to an accountant and steals their VPN credentials has chosen phishing as the attack vector — the admin panel was on the surface but was not the path used.

The practical lesson: reduce the surface (close unused ports, disable unused features), and monitor the vectors that attackers actually use (email, exposed web apps, credentials).

## Mapping attacks: kill chain and MITRE ATT&CK

Two frameworks help you reason about the stages of an attack.

### Cyber Kill Chain

Developed by Lockheed Martin, it models a targeted intrusion as seven sequential stages. Defending any early stage can stop the chain.

| # | Stage | Example activity | Example defence |
|---|---|---|---|
| 1 | Reconnaissance | Scanning networks, researching employees | Minimize public data, monitor exposed services |
| 2 | Weaponization | Packaging an exploit with a payload | Application patching, allow-listing |
| 3 | Delivery | Sending the phishing email | Email filtering, user awareness |
| 4 | Exploitation | Triggering the vulnerability | Patch management, sandboxing |
| 5 | Installation | Dropping a backdoor, persistence key | Endpoint detection, integrity monitoring |
| 6 | Command and control | Beaconing to a remote server | Egress filtering, DNS monitoring |
| 7 | Actions on objectives | Stealing or encrypting data | Data loss prevention, backups, segmentation |

### MITRE ATT&CK

ATT&CK is a public knowledge base of real-world attacker behaviour, organized by **tactics** (the attacker's goal) and **techniques** (how they achieve it). Major tactic categories include reconnaissance, resource development, initial access, execution, persistence, privilege escalation, defense evasion, credential access, discovery, lateral movement, collection, command and control, exfiltration, and impact.

Where the kill chain is a linear story, ATT&CK is a catalogue: defenders use it to map detected events to technique IDs (for example, T1566 for phishing), find gaps in detection, and communicate precisely about threats. Groups such as APT28 have entries listing the specific techniques they are observed using.

## Scenario

A mid-sized manufacturer is compromised. Walking the kill chain:

1. **Reconnaissance:** the attacker browses the careers page and learns the company uses a specific VPN appliance.
2. **Delivery:** an HR employee opens a job-application PDF that runs an exploit against an unpatched component.
3. **Installation and C2:** a small implant beacons to a legitimate cloud host every hour, blending into normal traffic.
4. **Actions on objectives:** after weeks of discovery and lateral movement, the attackers reach file servers and exfiltrate engineering drawings.

Each stage left signals — an unusual PDF, an odd outbound connection, abnormal file access — but no single alarm was enough. ATT&CK mapping would place these at T1566 (phishing), T1071 (application layer protocol C2), and T1005 (data from local system), revealing that detections existed for delivery but not for command and control or exfiltration.

## Prioritizing defences

You cannot defend everything at once, so order the work:

1. **Cover the high-volume vectors first:** email filtering and MFA against phishing, patching of internet-facing systems, and changed default credentials.
2. **Assume initial access will happen** at some point: invest in detection for persistence, credential access, and lateral movement — the middle stages of ATT&CK.
3. **Match effort to plausible actors.** A small business faces mostly cybercriminals and opportunists, so backups and MFA buy more than zero-day defence. A defence contractor must plan for nation-state persistence.
4. **Rehearse the response.** Knowing which log to check during an incident matters more than owning every possible tool.

## See also

- [The CIA Triad](cia-triad.md)
- [Malware Types](malware.md)
- [Social Engineering](social-engineering.md)
- [Network Security Basics](network-basics.md)
