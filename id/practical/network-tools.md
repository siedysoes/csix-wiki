# Mendalami Tool Jaringan

Halaman ini adalah referensi praktis untuk tool inti penyelidikan jaringan: Wireshark, nmap, netcat, utilitas lookup DNS, plus konsep di balik intercepting proxy. Jika digunakan dengan benar, tool-tool ini menjawab pertanyaan konkret tentang apa yang dilakukan sebuah sistem — dan jika digunakan sembarangan, beberapa di antaranya ilegal terhadap jaringan yang tidak Anda miliki.

## Catatan hukum lebih dulu

Hanya pindai, tangkap, dan teliti sistem serta jaringan yang Anda miliki atau yang secara tertulis Anda beri izin untuk diuji. Port scanning terhadap pihak ketiga dapat melanggar hukum kejahatan komputer di sebagian besar yurisdiksi, dan menangkap lalu lintas orang lain dapat melanggar hukum penyadapan. Dapatkan otorisasi, tetapkan cakupan, dan utamakan target lab/saat belajar.

## Wireshark

Wireshark menangkap dan membedah paket dari sebuah antarmuka. Lakukan capture dengan snaplen yang masuk akal, lalu saring — menganalisis semuanya itu lambat dan berisik.

Display filter yang berguna:

```text
tcp.port == 443              # HTTPS traffic on port 443
ip.addr == 10.0.0.5          # traffic to/from one host
http.request.method == "GET" # HTTP requests only
dns                          # DNS queries and responses
tcp.flags.syn == 1 and tcp.flags.ack == 0   # connection attempts
not arp                      # hide address resolution noise
```

**Mengikuti TCP stream**: klik kanan paket → *Follow* → *TCP Stream* (atau `Follow → HTTP Stream`). Ini merangkai ulang percakapan secara berurutan sehingga Anda dapat membaca request/response yang sebenarnya, termasuk header dan body untuk protokol yang tidak terenkripsi. Untuk lalu lintas TLS Anda sebagian besar hanya akan melihat rekaman terenkripsi kecuali Anda memiliki kunci sesi (misalnya melalui SSLKEYLOGFILE dari klien Anda sendiri).

Tips capture:

- Capture di antarmuka tempat lalu lintas benar-benar mengalir; gunakan port SPAN/mirror pada switch atau hub/tap untuk lalu lintas orang lain yang Anda amati dengan izin.
- Gunakan capture filter (BPF, misalnya `tcp port 443`) untuk membatasi volume, lalu display filter setelahnya untuk menyelidiki.
- Simpan file `.pcapng` sebagai bukti; file tersebut menyimpan komentar, info antarmuka, dan statistik drop.

## nmap

nmap adalah port scanner dan service enumerator. Mulai dengan scan SYN default yang aman terhadap target yang berotorisasi:

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

Flag utama:

| Flag | Tujuan |
| --- | --- |
| `-sn` | Host discovery saja (tanpa pemindaian port) |
| `-sV` | Menjajah port terbuka untuk mendeteksi layanan dan versi |
| `-O` | Deteksi OS (sering memerlukan root/Administrator) |
| `-p-` / `-p 22,80` | Semua port / port tertentu |
| `-T0`–`-T5` | Template timing; T4 agresif, T5 bisa membanjiri koneksi |
| `--open` | Tampilkan hanya port terbuka |
| `-oA out` | Simpan keluaran dalam semua format (normal, XML, grepable) |

Timing itu penting: T5 bisa menjatuhkan perangkat embedded yang rapuh dan terlihat seperti serangan di log. Untuk belajar, berlatihlah di lab Anda sendiri (lihat panduan home lab) atau rentang yang sengaja dibuat rentan.

## Dasar netcat

netcat ("pisau lipat Swiss-nya TCP/IP") membuka koneksi mentah dan bisa mendengarkan — berguna untuk tes konektivitas cepat dan transfer.

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

Catatan: lalu lintas netcat biasa tidak terenkripsi — jangan pernah mengirimkan data sensitif melaluinya di luar lab.

## curl untuk debugging HTTP

curl adalah cara tercepat untuk memeriksa perilaku HTTP tanpa browser.

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

Sangat berguna untuk memeriksa security header (`curl -I`), respons rate-limit, dan persis apa yang dikembalikan sebuah API.

## dig dan nslookup untuk penyelidikan DNS

Pertanyaan DNS — "nama ini resolve ke apa?", "siapa yang authoritative?", "apakah TXT record ini asli?" — dijawab dengan `dig` (lebih disukai) atau `nslookup`.

```bash
dig example.com A                 # basic A record query
dig +short example.com            # answer only
dig @1.1.1.1 example.com AAAA     # ask a specific resolver
dig example.com MX                # mail servers
dig example.com NS +trace         # walk delegation from the root
dig -x 93.184.216.34              # reverse lookup
nslookup example.com              # simpler alternative, less detail
```

Cara membaca jawaban: bagian `ANSWER` berisi record, `AUTHORITY` menyebut pemilik zona, dan `ADDITIONAL` mungkin memuat glue record. Respons terpotong atau SERVFAIL adalah petunjuk selama penyelesaian masalah atau penyelidikan cache poisoning.

## Konsep proxy tool (Burp / OWASP ZAP)

Intercepting proxy berada di antara browser Anda dan aplikasi target, memungkinkan Anda memeriksa dan memodifikasi request serta response. Ini adalah cara standar untuk menemukan kelemahan injection, autentikasi, dan access control di web app yang sedang Anda uji.

- **Burp Suite** (PortSwigger) dan **OWASP ZAP** (gratis, open source) sama-sama bekerja dengan mengarahkan HTTP proxy browser ke `127.0.0.1:8080` dan memasang sertifikat CA milik tool untuk mendekripsi HTTPS.
- Alur kerja khas: telusuri lewat proxy untuk memetakan aplikasi → repeat/replay request → kirim ke Repeater (Burp) / Manual Request Editor (ZAP) untuk melakukan fuzzing input → catat buktinya.
- Gunakan hanya terhadap aplikasi yang Anda beri izin untuk diuji (lab Anda, program bug bounty vendor, atau kontrak kerja tertulis). Tetapkan cakupan setiap sesi secara eksplisit.

## Pertanyaan → tool → contoh

| Pertanyaan | Tool | Contoh |
| --- | --- | --- |
| Apakah host ini hidup? | nmap | `nmap -sn 192.168.1.0/24` |
| Layanan/versi apa yang berjalan di 8443? | nmap | `nmap -sV -p 8443 host` |
| Lalu lintas apa yang sebenarnya dikirim klien? | Wireshark | filter `tcp.port == 443`, Follow TCP Stream |
| Apakah firewall yang memblokir saya atau layanan yang mati? | netcat | `nc -vz host 443` |
| Apa yang dikembalikan API ini dan secepat apa? | curl | `curl -v -w "%{http_code}" https://api/x` |
| Ke mana email domain ini dikirim? | dig | `dig example.com MX` |
| Apa yang dikirim aplikasi ke hulu? | Burp/ZAP | Proxy-kan browser, replay di Repeater |
| Apakah sebuah DNS record baru diubah belakangan ini? | dig | `dig example.com TXT +trace` |

## Lihat juga

- [Membangun Home Lab](hands-on-lab.md)
- [Dasar Keamanan Jaringan](../fundamentals/network-basics.md)
- [Dasar Incident Response](incident-response.md)
- [Keamanan Web & OWASP Top 10](../fundamentals/web-security-owasp.md)
