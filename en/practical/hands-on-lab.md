# Building a Home Lab

A home lab is a private, isolated environment — virtual machines, a virtual network, and security tooling — where you practice attacking and defending systems you own. It turns passive reading into practical skill, and it gives you a safe place to break things, recover, and explain what happened.

## Ethics and legality first

One rule governs everything else on this page: **only test systems you own or have explicit, written authorization to test.** Scanning, exploiting, or even connecting to a system without permission is a criminal offence in most jurisdictions, and the laws differ by country — "I was just practising" is not a defence.

- Keep all lab traffic inside lab-owned address space. The ranges `192.0.2.0/24`, `198.51.100.0/24`, and `203.0.113.0/24` are reserved for documentation and never route on the internet.
- Give every vulnerable target and attack tool an isolated network that cannot reach your home devices, your workplace, or the internet.
- Never run exploits against third-party machines "to see if it works", and never upload captured credentials or malware to public services.
- If you use a practice platform (TryHackMe, Hack The Box, OverTheWire), its scope defines what is authorized for you — stay inside it.
- For group labs, write down scope, allowed techniques, and an emergency contact before anyone starts.

If you are unsure whether something is authorized, stop and ask the owner in writing.

## Virtualization options

| Option | Runs on | Good for | Limits |
|---|---|---|---|
| VirtualBox | Windows, macOS, Linux | Free full VMs, beginner-friendly | Slower disk I/O than paid hypervisors |
| VMware Workstation Pro / Fusion | Windows, macOS, Linux | Better performance and snapshots (free for personal use) | Licensing for commercial use |
| WSL 2 | Windows | Linux tooling (nmap, tcpdump, Python) without a full VM | Not a security boundary; cannot host guest targets |
| Proxmox VE / ESXi | Bare-metal host | Longer-lived labs on a spare PC | Needs dedicated hardware |
| Docker | Any host with a container runtime | Lightweight web targets such as DVWA or Juice Shop | Containers share the host kernel; weaker isolation than VMs |

Rule of thumb: use VMs for targets you will attack and exploit, and WSL or containers for tooling and simple web targets.

## Isolated networks

Never bridge lab VMs onto your real LAN. Build a private network instead:

- **Host-only network** — VMs and the host can talk to each other, nothing else. In VirtualBox this creates an adapter such as `192.168.56.1`; ideal when your attack VM must reach a target VM but neither needs the internet.
- **Internal network** — VM-to-VM only, no host access. Use it for segments you want fully cut off.
- **NAT** — outbound internet for downloads. Use it only on the attack VM when you need package updates, then disconnect it.
- **No adapter (air gap)** — for malware analysis and capture of hostile traffic.

A common layout: attack VM on `192.168.56.0/24` (host-only) plus a second adapter on NAT for updates; target VMs get only the host-only adapter. Verify with `ip addr` on Linux or `ipconfig` on Windows before running anything.

## Snapshot strategy

Snapshots let you rewind a machine to a known state in seconds, which is what makes repeated practice possible.

1. Install the guest OS, apply updates, install baseline tools — then take a snapshot named `baseline` and never edit it again.
2. Take a named snapshot before each risky step: `pre-exploit`, `pre-patch`, `configured`.
3. Revert to `baseline` after every exercise so the next run starts from the same state. Reverting discards everything recorded after that snapshot, so intermediate snapshots are for short detours only.
4. Keep two or three snapshots per VM. They live on the host disk and grow as the VM writes; long chains break easily.

Snapshots are not backups. Keep real exports of anything you cannot rebuild from a clean install.

## Deliberately vulnerable targets

These projects exist to be broken — install them only inside the lab network.

| Target | What it teaches | Notes |
|---|---|---|
| DVWA (Damn Vulnerable Web Application) | SQL injection, XSS, CSRF, command injection | PHP app with difficulty levels from low to impossible |
| OWASP Juice Shop | Modern OWASP Top 10, API security, blind XSS | Node.js; often run as `docker run -d -p 3000:3000 bkimminich/juice-shop` |
| Metasploitable 2 / 3 | Deliberately weak Linux services, target for exploit frameworks | Lab only; never expose to any network you do not own |
| VulnHub images | Full vulnerable VMs with capture-the-flag style goals | Download and import into your hypervisor |
| OWASP WebGoat | Guided lessons on injection, access control, authentication | Runs from Docker or a local JVM |

Download only from official project pages or their GitHub/Docker Hub organizations; random "hacking VMs" from untrusted sites are a common malware delivery trick.

## Packet capture with Wireshark

Wireshark shows exactly what your tools do on the wire — the fastest way to understand TCP handshakes, DNS, and HTTP.

1. Select the correct interface: `Capture > Options`, then pick the host-only adapter (VirtualBox: `VirtualBox Host-Only Network`). The live traffic list helps if you are unsure.
2. Start the capture, then run your scan or browse the target.
3. Stop, and filter:
   - `tcp.port == 80` — HTTP traffic
   - `ip.addr == 192.168.56.10` — one host
   - `dns` — name lookups
   - `tcp.flags.syn == 1 and tcp.flags.ack == 0` — scan probes (SYN packets)
4. Right-click a packet, then Follow > TCP Stream to reassemble a conversation.

For headless hosts, `tcpdump -i any -w lab.pcap` writes a file Wireshark can open later. When a connection fails, capture from both ends (attacker and target) — the discrepancy usually reveals the cause.

## Basic nmap usage

`nmap` is the standard network scanner. Run it only against lab ranges.

```bash
# 1. Find live hosts on the lab subnet
nmap -sn 192.168.56.0/24

# 2. Full TCP port scan with service/version detection and default scripts
nmap -sV -sC -p- -T4 192.168.56.10

# 3. Check for known vulnerable services (NSE "vuln" script category)
nmap --script vuln 192.168.56.10

# 4. UDP is slow; scan the most common ports only
nmap -sU --top-ports 20 192.168.56.10
```

`-sV` fingerprints services, `-sC` runs default scripts, `-p-` scans all 65535 TCP ports. A `--script vuln` match is a lead, not proof — verify it manually. Save results with `-oA scan1` to get `.nmap`, `.xml`, and `.gnmap` files at once.

## Low-resource lab alternatives

- **One attack VM plus containers**: a single Kali or Parrot VM (2 vCPU, 4 GB RAM) with Docker targets keeps the total footprint under 6 GB.
- **WSL 2 on Windows**: run nmap, Python, and tcpdump for tool practice without any target VMs.
- **Browser-hosted labs**: TryHackMe and Hack The Box run the targets for you — zero local resources and always authorized.
- **Minimal guests**: Alpine Linux or Debian netinstall images for targets, hundreds of megabytes instead of multi-gigabyte desktop installs.
- **Rebuild from `baseline` instead of reinstalling**: a revert costs seconds and no disk space.

## Lab components table

| Component | Purpose | Typical choices |
|---|---|---|
| Hypervisor | Runs the VMs | VirtualBox, VMware Workstation Pro/Fusion, Proxmox VE |
| Attack machine | Scanning, exploitation, analysis | Kali Linux, Parrot OS, or plain Debian with tools added |
| Target machines | Systems you practice against | DVWA, Juice Shop, Metasploitable-style VMs, VulnHub images |
| Network | Isolation from the real world | Host-only and internal virtual networks |
| Capture and analysis | Observe traffic | Wireshark on host or attack VM, tcpdump on headless guests |
| Documentation | Reuse and explain your work | Lab notebook: IPs, credentials, commands, results |
| Snapshots | Fast, repeatable reset | Named snapshots, periodic export of clean VMs |

## First-weekend setup plan

**Saturday morning (2-3 hours)**

1. Install the hypervisor and a Linux attack VM from an official image; verify the published checksum.
2. Install guest additions, set a static lab IP, take the `baseline` snapshot.

**Saturday afternoon (2-3 hours)**

3. Create the host-only network `192.168.56.0/24`, attach only the attack VM, and confirm it reaches the host but not the internet.
4. Install Wireshark, capture on the host-only interface, and verify you can see your own traffic.

**Sunday morning (2-3 hours)**

5. Add one target VM (DVWA, or Juice Shop in Docker) with only the host-only adapter.
6. Run the four `nmap` commands above, save the output, and match open ports to what Wireshark captured.

**Sunday afternoon (1-2 hours)**

7. Break one thing on purpose (change a config, restart a service), revert to `baseline`, and confirm the reset works.
8. Write a one-page summary: topology, IPs, commands run, results observed. That page is your lab README and your interview evidence.

A lab you can rebuild in an hour from notes and snapshots is worth more than a lab you hand-crafted over a weekend and cannot reproduce.

## See also

- [Network Security Basics](../fundamentals/network-basics.md)
- [Operating System Security](../fundamentals/operating-system-security.md)
- [Web Security & the OWASP Top 10](../fundamentals/web-security-owasp.md)
- [Cybersecurity Learning Path](../career/learning-path.md)
