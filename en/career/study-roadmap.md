# Study Roadmap: 3 / 6 / 12 Months

This page turns the learning path into three concrete plans, depending on how much time you have and whether you already work in IT. Each table lists monthly goals, weekly habits, and a milestone or checkpoint you must pass before moving on — treat the checkpoints as gates, not suggestions.

## Choosing a timeline

| Timeline | Best for | What it can realistically deliver |
|----------|----------|------------------------------------|
| 3 months | People with existing IT or developer experience, studying full time | A solid foundation plus entry-cert preparation; a first job is possible but competition will be tough |
| 6 months | Career changers studying part time with a consistent schedule | Foundation, one entry certification, and a small portfolio |
| 12 months | Beginners or those combining study with a full-time job | Foundation, certification, specialization trial, portfolio and interview readiness |

### A workable week

| Slot | Activity |
|------|----------|
| One fixed daily slot (60-90 minutes) | Focused study on the current stage's topic, with notes in your own words |
| One weekend block (2-3 hours) | Hands-on lab, project or CTF work — uninterrupted practice time |
| Short daily slot (15-20 minutes) | Flashcard review and re-reading the previous session's notes |
| At the end of every session | A five-minute summary written from memory, added to your notes file |
| Once a month | Re-read all notes, update your portfolio, and check the current milestone honestly |

## 3-month plan (intensive)

| Month | Goals | Weekly habits | Milestone / checkpoint |
|-------|-------|---------------|-------------------------|
| 1 | IT fundamentals and networking core: TCP/IP model, subnetting, DNS, DHCP, HTTP/TLS, common ports | 8-10 hours of study; one Wireshark capture analyzed per week; daily terminology review | Trace a DNS query end to end from memory; solve subnetting drills without a calculator |
| 2 | Linux and Windows fundamentals plus Python scripting basics | Daily terminal session; one small automation script per week (log summarizer, port checker) | Administer a Linux VM entirely from the CLI; run a script that parses a log file and prints a summary |
| 3 | Security fundamentals, guided labs, entry certification prep | 3-5 guided lab rooms or equivalent; a short write-up after each completed challenge | Complete one beginner lab unaided end to end; score consistently on full practice question sets for your chosen entry cert |

## 6-month plan (balanced)

| Month | Goals | Weekly habits | Milestone / checkpoint |
|-------|-------|---------------|-------------------------|
| 1 | IT fundamentals: hardware, files, processes, virtualization, troubleshooting | 5-7 hours of study; build one VM and document its configuration | Explain every component of your own computer and recover a deliberately broken VM from a snapshot |
| 2 | Networking deep dive: addressing, routing, DNS, DHCP, TLS, HTTP, ports | Packet analysis twice a week; weekly flashcard review | Capture and explain a real HTTPS session: DNS, TCP handshake, TLS negotiation, HTTP request |
| 3 | Operating systems: Linux CLI, permissions, services, logs; Windows equivalents | Alternate Linux and Windows sessions; one hardening task per week | Set up users, permissions and a service on Linux, then locate its logs on both platforms |
| 4 | Scripting and automation with Python (plus Bash or PowerShell exposure) | One script per week that removes a manual step from your own workflow | Publish a working script with a README that a stranger can run without asking you questions |
| 5 | Security fundamentals: CIA triad, threats, malware, social engineering, crypto, auth, OWASP Top 10 | Scenario practice; begin the lab platform track for your chosen direction | Explain each OWASP Top 10 category with a concrete example and one mitigation |
| 6 | Guided labs, first portfolio outputs, specialization trial | 4-6 labs or CTF challenges; two published write-ups | Finish a beginner-to-intermediate lab streak and pick your first specialization |

## 12-month plan (thorough)

| Month | Focus | Weekly habits | Milestone / checkpoint |
|-------|-------|---------------|-------------------------|
| 1-2 | IT fundamentals and networking | 4-6 hours weekly; one packet capture analyzed per week | Subnetting, TCP/IP and DNS explained from memory; a written networking notes file you can revise |
| 3-4 | Operating systems: Linux and Windows administration | Alternate OS sessions; weekly log-hunting exercise | Administer both platforms from the command line and find the relevant logs for a given event |
| 5-6 | Scripting and automation | One Python project every two weeks; read other people's code weekly | Three small tools in a GitHub repository with READMEs |
| 7-8 | Security fundamentals plus entry certification | Scenario questions weekly; lab session weekly | Pass a full practice exam for your chosen entry cert, then schedule the real one |
| 9-10 | Hands-on labs and a specialization trial | 2-3 labs or CTF challenges weekly; one write-up per completed machine | Complete a coherent streak in one track (blue, red, AppSec, cloud or GRC) and explain why you chose it |
| 11 | Portfolio deepening and an intermediate credential or serious project | Weekly publication; review your own write-ups for gaps | A portfolio with 4+ write-ups, a GitHub project and a lab diagram; intermediate cert attempted if the timing fits |
| 12 | Interview preparation and applications | Two mock interviews or technical drills weekly; apply steadily | Answer networking and triage questions unprompted; walk a reviewer through your portfolio in 10 minutes |

## Building a home lab safely and legally

- **Isolate it**: use a hypervisor (VirtualBox, VMware Workstation Player or similar) with host-only or internal networking. Never bridge vulnerable VMs to your production network, and do not port-forward them to the internet.
- **Use trusted images**: official OS ISOs, intentionally vulnerable images from reputable projects, or virtual machines provided by lab platforms. Verify downloads against published checksums where available.
- **Snapshot first**: take a clean snapshot before every exercise and revert afterwards, so a broken experiment costs you nothing.
- **Test only what you own or are authorized to attack**: that includes your employer's systems — written scope matters. Do not scan or attack public IP ranges "for practice"; use lab platform VPN ranges, which are explicitly in scope.
- **Keep deliberate weaknesses contained**: vulnerable images are insecure by design, so run them offline or on an isolated network and never reuse their credentials elsewhere.

### Starter build

A first home lab needs very little hardware: one hypervisor, a Linux VM for tooling, a second Linux VM as a defender's target, one intentionally vulnerable VM from a reputable source, and snapshots of each clean state. Add a Windows VM later if your goal needs it, using an evaluation or your own licensed copy. Keep the attack range on a host-only network, write down every change you make, and take a new snapshot each time the lab reaches a known-good state.

## Portfolio and proof of work

- **Lab and CTF write-ups**: methodology, commands used, what failed first, what you learned. Avoid spoilers for active challenges, and never publish material that violates a platform's rules.
- **GitHub projects**: small tools that actually run — a log summarizer, a security-header checker, a wordlist organizer — each with a README, install steps and a sample output.
- **Detection content**: for blue team direction, publish rules (for example Sigma-style detections) derived from behaviors you observed in labs, with a short explanation of the log source.
- **Lab documentation**: a network diagram of your home lab plus a hardening checklist you applied, which doubles as interview material.
- **A short blog series**: explain one concept you had to struggle through; explaining is how you find out what you do not understand.

## Interview prep

- **Behavioral**: prepare STAR-format answers for why security, a project that failed, a conflict in a team, how you keep learning, and how you handle ambiguity or tight deadlines. Every bullet on your resume must be explainable in two minutes.
- **Technical (defensive)**: walk through triaging a sample alert — which logs and systems you check first, how you separate an indicator from a false positive, and how you would document the incident. Refresh DNS, the TCP handshake, TLS and common log sources.
- **Technical (offensive)**: describe your methodology for a host or web engagement, your enumeration order, what you do when you are stuck, and how you decide what goes into a report.
- **Practical**: expect a screen-share task such as reading a log snippet or writing a short script. Practice narrating your thought process while you work.
- **Sample technical drills**: why DNS matters when investigating a malicious download; what happens during an HTTPS connection from DNS to TLS; what you check first when a single host alerts on outbound traffic to an unusual port; how you would prove an alert was a false positive.
- **Questions for them**: ask about the team's tooling, on-call or escalation process, and how success is measured in the first six months.

## Common pitfalls

| Pitfall | What it looks like | Fix |
|---------|--------------------|-----|
| Tutorial hell | Endless courses, nothing built unaided | Close the tutorial and rebuild the same result from scratch, then document it |
| Cert collecting without skills | A stack of vouchers but cannot explain subnetting or read a log | One cert at a time, skill first, exam second |
| Skipping networking basics | Cannot interpret a packet capture or explain DNS resolution | Return to the networking stage before buying anything else |
| Theory only, no labs | Correct definitions, zero hands-on hours | Schedule lab time in the calendar with the same weight as reading |
| Undocumented work | Nothing concrete to show in interviews | Publish a write-up or repo after every completed lab |
| Unauthorized testing | Scanning networks or hosts outside written scope | Practice only on your lab or in-scope platform environments |

## When to adjust the plan

- Missed the same checkpoint twice in a row: slow down and repeat the previous month's goals instead of pushing forward.
- Finished a month early: deepen the labs, do not add extra certificates to fill the time.
- Schedule collapsed because of work, exams or family duties: cut weekly hours before cutting consistency — a reduced plan beats an abandoned one.
- Realized the chosen specialization does not fit you: switch during a plan's trial months; discovering it in month 2 costs far less than in month 12.

## See also

- [Cybersecurity Learning Path](learning-path.md)
- [Security Certifications Guide](certifications.md)
- [Network Security Basics](../fundamentals/network-basics.md)
- [Operating System Security](../fundamentals/operating-system-security.md)
