# Networking Tools Deep-Dive

This page is a hands-on reference for the core network investigation tools: Wireshark, nmap, netcat, curl, and DNS lookup utilities, plus the concept behind intercepting proxies. Used correctly, these tools answer concrete questions about what a system is doing — and used carelessly, several of them are illegal against networks you do not own.

## Legal note first

Only scan, capture, and probe systems and networks you own or have explicit written authorization to test. Port scanning a third party can violate computer crime laws in most jurisdictions, and capturing others' traffic can violate wiretapping laws. Get authorization, define scope, and prefer test/lab targets while learning.

## Wireshark

Wireshark captures and dissects packets from an interface. Capture with a sensible snaplen, then filter — analyzing everything is slow and noisy.

Useful display filters:

```text
tcp.port == 443              # HTTPS traffic on port 443
ip.addr == 10.0.0.5          # traffic to/from one host
http.request.method == "GET" # HTTP requests only
dns                          # DNS queries and responses
tcp.flags.syn == 1 and tcp.flags.ack == 0   # connection attempts
not arp                      # hide address resolution noise
```

**Following a TCP stream**: right-click a packet → *Follow* → *TCP Stream* (or `Follow → HTTP Stream`). This reassembles the conversation in order so you can read the actual request/response, including headers and bodies for unencrypted protocols. For TLS traffic you will mostly see encrypted records unless you have the session keys (e.g., via SSLKEYLOGFILE from your own client).

Capture tips:

- Capture on the interface where traffic actually flows; use a switch SPAN/mirror port or a hub/tap for others' traffic you are authorized to observe.
- Use a capture filter (BPF, e.g. `tcp port 443`) to limit volume, display filters afterward to investigate.
- Save `.pcapng` files as evidence; they store comments, interface info, and drop statistics.

## nmap

nmap is a port scanner and service enumerator. Start with the safe, default-syn scan against authorized targets:

```bash
# Top ports, service/version detection, no scripts, normal timing
nmap -sV --top-ports 100 -T4 192.168.1.10

# Ping sweep first to find live hosts on a subnet you own
nmap -sn 192.168.1.0/24

# Full TCP connect scan (works without raw socket privileges, noisier)
nmap -sT -p 1-1000 192.168.1.10

# Slow and polite on fragile devices
nmap -T2 -sV 192.168.1.20
```

Key flags:

| Flag | Purpose |
| --- | --- |
| `-sn` | Host discovery only (no port scan) |
| `-sV` | Probe open ports to detect service and version |
| `-O` | OS detection (often needs root/Administrator) |
| `-p-` / `-p 22,80` | All ports / specific ports |
| `-T0`–`-T5` | Timing template; T4 is aggressive, T5 can overwhelm links |
| `--open` | Show only open ports |
| `-oA out` | Save output in all formats (normal, XML, grepable) |

Timing matters: T5 can crash fragile embedded devices and looks like an attack in logs. For learning, practice against your own lab (see the home lab guide) or intentionally vulnerable ranges.

## netcat basics

netcat ("the TCP/IP Swiss army knife") opens raw connections and can listen — useful for quick connectivity tests and transfers.

```bash
# Is a port open? (connection succeeded means open)
nc -vz 192.168.1.10 443

# Grab a banner
nc 192.168.1.10 22

# Simple listener on port 9001 (reverse-shell drills: use only in your lab)
nc -lvnp 9001

# Send a file between two lab machines
nc -lvnp 9001 < payload.tar.gz        # receiver
nc 192.168.1.20 9001 > payload.tar.gz  # sender
```

Note: plain netcat traffic is unencrypted — never carry sensitive data over it outside a lab.

## curl for HTTP debugging

curl is the fastest way to inspect HTTP behavior without a browser.

```bash
# Show response headers only
curl -I https://example.com

# Verbose: DNS, TCP, TLS handshake, headers, and body
curl -v https://example.com

# Send a JSON POST with a custom header
curl -X POST https://api.example.com/login \
  -H "Content-Type: application/json" \
  -d '{"user":"demo","pass":"test"}' \
  -w "\nHTTP %{http_code} in %{time_total}s\n"

# Follow redirects and show the final URL
curl -Ls -o /dev/null -w "%{url_effective}\n" https://example.com

# Test an internal API without proxy interference
curl --noproxy '*' http://127.0.0.1:8080/health
```

Handy for checking security headers (`curl -I`), rate-limit responses, and exactly what an API returns.

## dig and nslookup for DNS investigation

DNS questions — "what does this name resolve to?", "who is authoritative?", "is this TXT record real?" — are answered with `dig` (preferred) or `nslookup`.

```bash
dig example.com A                 # basic A record query
dig +short example.com            # answer only
dig @1.1.1.1 example.com AAAA     # ask a specific resolver
dig example.com MX                # mail servers
dig example.com NS +trace         # walk delegation from the root
dig -x 93.184.216.34              # reverse lookup
nslookup example.com              # simpler alternative, less detail
```

Reading the answer: the `ANSWER` section holds records, `AUTHORITY` names the zone owner, and `ADDITIONAL` may include glue. Truncated or SERVFAIL responses are clues during troubleshooting or cache-poisoning investigations.

## Proxy tools concept (Burp / OWASP ZAP)

An intercepting proxy sits between your browser and the target application, letting you inspect and modify requests and responses. It is the standard way to find injection, authentication, and access-control flaws in web apps you are testing.

- **Burp Suite** (PortSwigger) and **OWASP ZAP** (free, open source) both work by pointing the browser's HTTP proxy at `127.0.0.1:8080` and installing the tool's CA certificate to decrypt HTTPS.
- Typical workflow: proxy browsing to map the app → repeat/replay requests → send to Repeater (Burp) / Manual Request Editor (ZAP) to fuzz inputs → note evidence.
- Use only against applications you are authorized to test (your lab, a vendor bug bounty program, or a written engagement). Scope every session explicitly.

## Question → tool → example

| Question | Tool | Example |
| --- | --- | --- |
| Is this host up? | nmap | `nmap -sn 192.168.1.0/24` |
| What service/version runs on 8443? | nmap | `nmap -sV -p 8443 host` |
| What traffic does the client actually send? | Wireshark | filter `tcp.port == 443`, Follow TCP Stream |
| Is the firewall blocking me or is the service down? | netcat | `nc -vz host 443` |
| What does this API return and how fast? | curl | `curl -v -w "%{http_code}" https://api/x` |
| Where does this domain's mail go? | dig | `dig example.com MX` |
| What is the app sending upstream? | Burp/ZAP | Proxy the browser, replay in Repeater |
| Was a DNS record changed recently? | dig | `dig example.com TXT +trace` |

## See also

- [Building a Home Lab](hands-on-lab.md)
- [Network Security Basics](../fundamentals/network-basics.md)
- [Incident Response Basics](incident-response.md)
- [Web Security & the OWASP Top 10](../fundamentals/web-security-owasp.md)
