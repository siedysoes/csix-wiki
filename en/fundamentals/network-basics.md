# Network Security Basics

Network security is the practice of protecting the confidentiality, integrity, and availability of data as it moves across local and wide area networks. Almost every attack path — from malware delivery to data exfiltration — depends on some network behavior, so understanding protocols, ports, and defensive controls is a required foundation for any security role.

## The OSI and TCP/IP Models

You do not need to memorize every layer, but you should be able to place common protocols and attacks at the right level.

| Layer | OSI name | Examples | Typical security concerns |
|---|---|---|---|
| 1-2 | Physical / Data Link | Ethernet, Wi-Fi, ARP | Rogue devices, MAC spoofing, evil twin APs |
| 3 | Network | IP, ICMP, routing | IP spoofing, packet sniffing, rogue routers |
| 4 | Transport | TCP, UDP | Port scanning, SYN floods, RST injection |
| 5-6 | Session / Presentation | SMB, TLS, SSH | Protocol downgrade, session hijacking |
| 7 | Application | HTTP, DNS, SMTP, FTP | Injection, phishing, credential theft |

The practical TCP/IP model collapses these into four layers (Link, Internet, Transport, Application). When you run `nmap` you are mostly working at layers 4 and 7; when you analyze a PCAP you may drop to layer 3.

## Common Ports and Protocols

| Port | Protocol | Purpose | Security notes |
|---|---|---|---|
| 21 | FTP | File transfer | Cleartext credentials; prefer SFTP (FTP over SSH) |
| 22 | SSH | Remote shell / secure transfer | Key-based auth, disable root password login |
| 25 | SMTP | Email relaying | Relay abuse for spam; restrict outbound where possible |
| 53 | DNS | Name resolution | Often abused for tunneling and data exfiltration |
| 80 | HTTP | Web | Cleartext; redirect to HTTPS |
| 443 | HTTPS | Web over TLS | Inspect with care; encrypted malware C2 hides here |
| 3389 | RDP | Remote Desktop | Frequent brute-force target; never expose directly to the internet |
| 445 | SMB | Windows file sharing | EternalBlue (MS17-010) era wormable service |
| 3306 / 5432 | MySQL / PostgreSQL | Databases | Should never be internet-facing |

Rule of thumb: every open port is a decision that needs a business justification, not a default.

## DNS and How It Is Abused

DNS translates names to IP addresses through a recursive resolver that queries authoritative servers. Because nearly all applications depend on it and it is usually allowed through firewalls, DNS is a favorite channel for attackers:

- **DNS tunneling** — encoding command-and-control or stolen data inside DNS queries (TXT, CNAME records).
- **Domain generation algorithms (DGAs)** — malware computes random domain names daily so blocklists go stale.
- **DNS hijacking / cache poisoning** — redirecting victims to attacker-controlled servers.
- **Fast-flux** — rotating DNS records rapidly to keep malicious infrastructure one step ahead of blocklists.

Defensive moves: enforce DNSSEC validation, force internal clients to use your own resolver (not arbitrary public DNS), log query volume and anomalies, and consider DNS filtering for known-malicious domains.

## Firewalls

### Packet-filtering firewall

Examines each packet's source/destination IP, protocol, and port against rules. Fast and cheap, but blind to connection context and to what is inside the payload.

| Type | Inspects | Strengths | Weaknesses |
|---|---|---|---|
| Packet filter (stateless) | Headers only | Very fast | No session state, easy to bypass |
| Stateful firewall | Headers + connection state | Tracks sessions | Still blind to application content |
| Next-generation (NGFW) | Adds DPI, TLS inspection | App awareness, IPS integration | Cost, decryption complexity |
| WAF | HTTP/HTTPS requests | Blocks web attacks (SQLi, XSS) | Only covers web traffic, needs tuning |

### Web Application Firewall (WAF)

A WAF sits in front of web applications and applies rules to HTTP requests — for example, rejecting a query string containing `UNION SELECT`. It is a compensating control, not a substitute for writing safe code: attackers encode, chunk, and obfuscate payloads to test bypasses.

Example rule intent (pseudocode, not vendor syntax):

```
IF http.request.uri.query CONTAINS "' OR 1=1--" THEN BLOCK
IF http.request.header.user-agent IS EMPTY THEN LOG
```

## VPN vs Proxy

| Aspect | VPN | Proxy |
|---|---|---|
| Scope | Routes all device traffic (or a whole network) | Routes one application or request |
| Encryption | Usually full tunnel with strong crypto | Depends; many proxies are plain HTTP CONNECT |
| Identity | Authenticate the device/user | Often anonymous, but the proxy operator sees everything |
| Typical use | Remote access to corporate resources, privacy on untrusted Wi-Fi | Content filtering, caching, simple anonymity |
| Trust | You trust the VPN provider with your metadata | You trust the proxy with the full request |

Neither makes you anonymous by default. A VPN shifts who can see your traffic; a proxy shifts who makes the request on your behalf.

## IDS and IPS

An **IDS** (Intrusion Detection System) monitors a copy of network traffic and raises alerts. An **IPS** (Intrusion Prevention System) sits inline and can drop malicious packets before they reach the target.

- Signature-based: fast, precise, but blind to new attacks.
- Anomaly/behavior-based: catches novel activity, but produces false positives that need tuning.
- Placement matters: an IDS on aSPAN port only sees traffic that is mirrored to it; an IPS fails open or closed depending on configuration during an outage.

Alerts are only useful with triage. A common interview question is whether an alert on `ET EXPLOIT` means a compromise — the answer is that it means something matched a pattern, and you still need to verify context.

## Network Segmentation

Segmentation limits lateral movement by dividing the network into zones with controlled traffic between them. If an attacker compromises a web server, segmentation stops them from reaching the finance VLAN directly.

Practical steps:

1. Inventory assets and classify data (public, internal, confidential).
2. Group by function and sensitivity: users, servers, databases, IoT/OT, management.
3. Allow only required flows between zones (default deny).
4. Protect administrative paths with jump hosts or privileged access workstations.
5. Monitor east-west traffic, not just north-south.

Microsegmentation takes this to the workload level, often enforced by host firewalls or cloud security groups rather than physical switches.

## Zero Trust: A Short Introduction

Zero trust replaces the old "trust the internal network" assumption with the principle: **never trust, always verify**. Core ideas include:

- Authenticate and authorize every request, regardless of source network.
- Use device health and context (location, patch level) in access decisions.
- Grant least-privilege, short-lived access instead of standing privileges.
- Assume breach: log everything, segment everything, be ready to contain.

Zero trust is a program, not a product. Buying a "zero trust appliance" without identity, segmentation, and logging maturity does not deliver zero trust.

## Scenario: Triage a Suspicious Outbound Connection

You see a workstation making DNS queries to unfamiliar domains every 60 seconds:

```
$ ps -ef | grep -i updater
$ sudo lsof -i -P -n | grep ESTABLISHED
updater  1843  user  4u  IPv4  55211  0t0  TCP 10.0.4.23:51204->203.0.113.77:443 (ESTABLISHED)
$ dig +short suspicious-example.xyz
203.0.113.77
```

Actions: isolate the host from the network, capture volatile evidence, check EDR telemetry, and block the domain/IP at the resolver and perimeter while you investigate. The repeated pattern plus low-reputation domain is a strong indicator of C2 beaconing.

## See also

- [The CIA Triad](cia-triad.md)
- [Cryptography Basics](cryptography-basics.md)
- [Web Security & the OWASP Top 10](web-security-owasp.md)
- [Operating System Security](operating-system-security.md)
- [Cybersecurity Learning Path](../career/learning-path.md)
