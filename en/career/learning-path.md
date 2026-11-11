# Cybersecurity Learning Path

Moving into security is easier when you build on general IT knowledge instead of jumping straight to tools. This page describes the stages from absolute beginner to job-ready, the main specialization areas you can choose afterwards, and where to practice — it is a sequence to follow, not a checklist to rush.

## Stage overview

| # | Stage | What you should be able to do | Rough pace (part time) |
|---|-------|-------------------------------|------------------------|
| 1 | IT fundamentals | Explain how hardware, operating systems, files and processes work; troubleshoot basic problems | 2-3 weeks |
| 2 | Networking | Subnetting, the TCP/IP model, DNS, DHCP, HTTP/HTTPS, common ports; read a packet capture | 4-6 weeks |
| 3 | Operating systems | Work in the Linux terminal and the Windows command line; manage users, permissions, services and logs | 3-4 weeks |
| 4 | Scripting | Write small Python (plus Bash or PowerShell) scripts to automate repetitive tasks | 3-4 weeks |
| 5 | Security fundamentals | Apply the CIA triad, recognize malware and social engineering, explain basic cryptography and authentication | 3-4 weeks |
| 6 | Hands-on labs | Solve guided rooms, CTF challenges and intentionally vulnerable machines | continuous |
| 7 | Specialization | Go deep in one track while keeping working knowledge of the others | ongoing |

The durations are indicative; the order matters more than the speed. Do not move on until you can explain the current stage without notes.

### Self-test gates

| Stage | You are ready to move on when you can... |
|-------|-------------------------------------------|
| 1 | Install an OS in a VM, recover files from a broken setup, and explain what a process is |
| 2 | Subnet a small network from memory and explain a packet capture line by line |
| 3 | Add users, set permissions, start a service and locate its logs on both Linux and Windows |
| 4 | Write a script that reads a file, filters lines and prints a summary without copying a tutorial |
| 5 | Explain the CIA triad, three malware categories and one OWASP Top 10 issue with mitigations |
| 6 | Complete a lab unaided, then write up what you did and why it worked |
| 7 | Describe your chosen track's day-to-day work and name the first job title you will apply for |

### Stage notes

- **IT fundamentals**: know what a CPU, disk, memory and process are; understand virtualization well enough to run a virtual machine; be comfortable installing software and reading error messages.
- **Networking**: the stage most often skipped and most often regretted. Focus on the TCP/IP model, addressing and subnetting, DNS, DHCP, TLS, HTTP and common ports, then practice with a packet analyzer such as Wireshark.
- **Operating systems**: Linux appears in most security tooling, Windows appears in most audits. Learn the command line, file permissions, users and groups, services, and where logs live on each.
- **Scripting**: Python is the usual first choice because its standard library covers file handling, parsing and HTTP. Automate any task you have already done by hand twice.
- **Security fundamentals**: study the concepts before the tools — the CIA triad, threat actors, malware categories, social engineering, cryptography basics, passwords and authentication, and the OWASP Top 10 for web.
- **Hands-on labs**: theory without practice does not survive an interview. Use deliberately vulnerable targets only in lab environments you are authorized to test.

## Specialization map

| Track | What they do day to day | Typical first role |
|-------|------------------------|--------------------|
| Blue team / SOC | Monitor alerts, triage incidents, tune detections and write reports or detection rules, often with a SIEM | SOC analyst / junior security analyst |
| Red team / pentest | Test systems with written permission, find and prove vulnerabilities, write findings and recommendations | Junior penetration tester / security consultant |
| Application security | Review code, threat-model features and run security testing within the development lifecycle | AppSec analyst / junior security engineer |
| Cloud security | Secure cloud accounts, identity and access, storage and networks; hunt misconfigurations | Cloud security analyst (often after a sysadmin or cloud admin role) |
| GRC | Policies, risk assessments, audits and compliance work against frameworks such as NIST and ISO 27001 | GRC / risk / compliance analyst |

You do not have to choose on day one. Run one beginner project from two or three tracks, notice which work you would happily repeat, then commit — but by month six of your studies you should be leaning in one direction, because job ads reward focus.

Whichever track you pick, the first four stages are shared ground: a blue teamer who cannot read a packet capture and a red teamer who cannot read logs both stall in interviews. Specialize late, not early — keep a working knowledge of the tracks next to yours, because AppSec people need to understand deployment and GRC people need to understand risk.

## Resources by category

### Official documentation and references

- NIST Computer Security Resource Center — public standards and guidance used worldwide
- CISA — advisories, alerts and free defensive guidance
- OWASP — Testing Guide, Cheat Sheets and the Top 10; essential for web and AppSec work
- MITRE ATT&CK — the shared knowledge base for describing attacker behavior
- Vendor docs you will actually use: Microsoft Learn, your Linux distribution's documentation, Python's official tutorial

### Lab platforms

- **TryHackMe** — guided, browser-based rooms that teach step by step; the gentlest entry point for beginners
- **Hack The Box** — machines and challenges that assume more independence; good once guided rooms start feeling easy
- **OverTheWire** — free wargames that drill Linux, shell and security fundamentals with no installation required

### Capture the flag (CTF)

- **picoCTF** — beginner-friendly and often run as a school or community event; a good first CTF
- **CTFtime** — calendar and scoreboard of public CTF events worldwide, plus archives of published write-ups
- Local and online club events — many communities run practice events you can join without travel

### Staying current

- Subscribe to one advisory feed (CISA or your national CERT) and read it weekly.
- Follow one reputable security news outlet and skim the headlines daily.
- Join one community — a local meetup, an online CTF team, or a forum — and read more than you post.

Pick one lab platform and one reference set, and finish something on them before opening ten browser tabs. A single completed room with a written summary teaches more than a bookmark folder of half-started courses, and course names change often — stick to the platform and documentation links above rather than any specific third-party course.

## Study method

- Take notes in your own words; copied notes are rarely read again.
- Use active recall: close the material and answer questions before re-reading it.
- Space reviews across days instead of cramming everything into one block.
- Write short explanations as if teaching someone else — the gaps become obvious.
- Track hours and completed labs in one file so your progress is measurable rather than felt.

## What job-ready looks like

- You can explain networking, operating system and security concepts from memory, without notes.
- You have completed a run of labs in one area and can describe your methodology, not just the results.
- You have something public to show: write-ups, a GitHub repository, or a documented home lab.
- You can perform a basic task while someone watches — read a log, trace a DNS query, or write a short script — and narrate your reasoning.
- You can name the specific first role you are applying for and match your evidence to its requirements.

## See also

- [Network Security Basics](../fundamentals/network-basics.md)
- [Operating System Security](../fundamentals/operating-system-security.md)
- [Web Security & the OWASP Top 10](../fundamentals/web-security-owasp.md)
- [Security Certifications Guide](certifications.md)
