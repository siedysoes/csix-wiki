# The CIA Triad

The CIA triad is a model of three security goals — confidentiality, integrity, and availability — used to define requirements and explain why incidents matter. Almost every control you will encounter, from disk encryption to off-site backups, exists to protect at least one of these three properties.

## Confidentiality

Confidentiality means only authorized parties can read the data. It is about preventing disclosure, not just to attackers but also to unauthorized employees, advertisers, or anyone who intercepts traffic in transit.

Real-world examples:

- Encryption at rest (BitLocker, LUKS) and in transit (TLS) so captured data is unreadable without keys.
- Access control: role-based permissions, least privilege, "need to know", multi-factor authentication.
- Data classification policies that mark customer records as restricted and limit who can export them.

**How a breach violates it:** attackers steal a database and publish it. The well-known Equifax breach of 2017 exposed personal data of roughly 147 million people — the data was disclosed to parties who had no right to see it, a pure confidentiality failure.

**Trade-offs:** strong confidentiality can reduce availability — if the only copy of an encryption key is lost, the data is gone. Strict need-to-know rules slow down collaboration, and end-to-end encryption conflicts with organizations that want to scan content for malware or compliance.

## Integrity

Integrity means data is accurate, complete, and modified only by authorized parties in authorized ways. It covers both unauthorized modification and unauthorized destruction of data.

Real-world examples:

- Hashing and digital signatures to detect tampering (software update signatures, SHA-256 checksums of downloads).
- File integrity monitoring tools (AIDE, Tripwire) that alert when system files change unexpectedly.
- Version control, audit logs, database constraints, and change-review processes that make unauthorized edits visible.

**How a breach violates it:** an attacker modifies transaction records to redirect payments, or edits log files to erase traces of an intrusion. Ransomware also damages integrity — encrypted files are corrupted beyond use even though no plaintext was leaked.

**Trade-offs:** integrity controls add friction. Four-eyes approvals, code review, and immutable audit logs slow delivery. Immutability also conflicts with privacy regulations that give people a right to erasure.

## Availability

Availability means authorized users can access systems and data when they need them. A perfectly confidential and intact system that nobody can reach during a business-critical moment has failed.

Real-world examples:

- Redundancy: RAID arrays, database replicas, failover clusters, multiple availability zones.
- Backups following the 3-2-1 rule (3 copies, 2 media, 1 off-site) with regularly tested restores.
- DDoS mitigation, autoscaling, load balancing, and timely patching to prevent outages.

**How a breach violates it:** a DDoS flood knocks a site offline, or ransomware encrypts every workstation — WannaCry in 2017 made hospitals unable to access patient records for days. Hardware failure without a tested backup is availability loss without any attacker at all.

**Trade-offs:** high availability costs money and operational complexity. Strict patching may require maintenance downtime, aggressive rate limiting can block legitimate users, and synchronous replication across regions improves availability while increasing latency.

## Trade-offs between the three

Security decisions usually balance the three goals against each other.

| Decision | Confidentiality | Integrity | Availability |
|---|---|---|---|
| Full-disk encryption on laptops | Stronger | Neutral | Weaker if keys are lost |
| Read-only replicas for databases | Neutral | Stronger (single write source) | Stronger (failover) but writes can stall |
| Air-gapped offline backups | Stronger (no network reach) | Stronger (tamper-resistant) | Weaker (restore takes time) |
| Mandatory approval before production changes | Neutral | Stronger | Slightly weaker (slower incident fixes) |
| Rate limiting / bot protection | Neutral | Neutral | Stronger for the service, weaker for legitimate heavy users |

A useful habit: for any control you design or review, ask which of the three it protects, which it weakens, and whether that balance matches the risk.

## Scenario

A small online shop evaluates three proposals for its customer database:

1. Store it on a single office PC, no backups. Cheap, but any disk failure or ransomware event destroys the business.
2. Put it in a public cloud bucket with world-readable access so the marketing team can "just grab what they need". Fast to work with, but confidentiality is gone.
3. Encrypt the database, restrict access to two roles, log every query, replicate to a second region, and take nightly backups with a monthly restore test.

Proposal 3 is the balanced one: confidentiality (encryption, access control), integrity (query logging, restricted write paths), and availability (replication, tested backups) are all addressed, with known and accepted costs.

## Testing the three properties

Controls are only real if you verify them.

| Property | How to test it |
|---|---|
| Confidentiality | Review access permissions, attempt reads as an unauthorized user, confirm encryption on stored and transmitted data |
| Integrity | Compare file hashes against known-good values, review audit logs for unexpected changes, practice a restore from backup |
| Availability | Run a failover drill, time a full backup restore, load-test the service, simulate an outage |

## Common mistakes

- Treating the triad as three checkboxes instead of a balance — one control can strengthen a goal while weakening another.
- Assuming confidentiality implies integrity: encrypted data can still be altered unless it is also authenticated (see authenticated encryption).
- Equating availability with speed: a slow system that eventually responds may still miss the business requirement.
- Testing once and never again: backups rot, permissions accumulate, and failover paths silently break.

## Related models

The CIA triad is a starting point, not the only model:

- **Parkerian hexad** (Donn Parker) extends the triad with possession, authenticity, and utility.
- **The DAD triad** (Disclosure, Alteration, Destruction) describes what an attacker achieves — the offensive mirror of CIA.
- **Privacy models** add lawful-use and consent concerns that CIA alone does not capture.
- Frameworks such as **NIST CSF** and **ISO 27001** organize controls around CIA-like outcomes.

## See also

- [Threat Landscape](threat-landscape.md)
- [Cryptography Basics](cryptography-basics.md)
- [Web Security & the OWASP Top 10](web-security-owasp.md)
- [Operating System Security](operating-system-security.md)
