# Dasar Keamanan Jaringan

Keamanan jaringan adalah praktik melindungi kerahasiaan, integritas, dan ketersediaan data saat mengalir melalui jaringan lokal maupun luas. Hampir setiap jalur serangan — dari pengiriman malware sampai ekfiltrasi data — bergantung pada perilaku jaringan tertentu, jadi memahami protokol, port, dan kontrol pertahanan merupakan fondasi wajib bagi setiap peran di bidang keamanan.

## Model OSI dan TCP/IP

Anda tidak perlu menghafal setiap layer, tetapi Anda harus mampu menempatkan protokol dan serangan yang umum pada level yang tepat.

| Layer | Nama OSI | Contoh | Kekhawatiran keamanan yang lazim |
|---|---|---|---|
| 1-2 | Physical / Data Link | Ethernet, Wi-Fi, ARP | Perangkat jahat (rogue), MAC spoofing, AP evil twin |
| 3 | Network | IP, ICMP, routing | IP spoofing, packet sniffing, router jahat |
| 4 | Transport | TCP, UDP | Port scanning, SYN flood, injeksi RST |
| 5-6 | Session / Presentation | SMB, TLS, SSH | Penurunan protokol (downgrade), session hijacking |
| 7 | Application | HTTP, DNS, SMTP, FTP | Injection, phishing, pencurian kredensial |

Model TCP/IP praktis merangkum lapisan-lapisan itu menjadi empat layer (Link, Internet, Transport, Application). Saat Anda menjalankan `nmap`, Anda sebagian besar bekerja di layer 4 dan 7; saat menganalisis PCAP, Anda bisa turun ke layer 3.

## Port dan Protokol yang Umum

| Port | Protokol | Fungsi | Catatan keamanan |
|---|---|---|---|
| 21 | FTP | Transfer file | Kredensial teks jelas; gunakan SFTP (FTP di atas SSH) |
| 22 | SSH | Remote shell / transfer aman | Autentikasi berbasis kunci, nonaktifkan login root dengan password |
| 25 | SMTP | Relay email | Penyalahgunaan relay untuk spam; batasi luar keluar bila memungkinkan |
| 53 | DNS | Resolusi nama | Sering disalahgunakan untuk tunneling dan ekfiltrasi data |
| 80 | HTTP | Web | Teks jelas; arahkan ke HTTPS |
| 443 | HTTPS | Web di atas TLS | Inspeksi dengan hati-hati; C2 malware terenkripsi bersembunyi di sini |
| 3389 | RDP | Remote Desktop | Sasaran brute-force yang sering; jangan pernah diekspos langsung ke internet |
| 445 | SMB | Berbagi file Windows | Layanan yang bisa dijangkiti worm sejak era EternalBlue (MS17-010) |
| 3306 / 5432 | MySQL / PostgreSQL | Basis data | Tidak boleh pernah menghadap internet |

Aturan praktis: setiap port terbuka adalah keputusan yang memerlukan justifikasi bisnis, bukan sekadar default.

## DNS dan Cara Penyalahgunaannya

DNS menerjemahkan nama menjadi alamat IP melalui resolver rekursif yang menanyakan server otoritatif. Karena hampir semua aplikasi bergantung padanya dan biasanya diizinkan melewati firewall, DNS menjadi kanal favorit penyerang:

- **DNS tunneling** — mengkodekan command-and-control atau data curian di dalam query DNS (record TXT, CNAME).
- **Domain generation algorithms (DGA)** — malware menghitung nama domain acak setiap hari sehingga blocklist menjadi usang.
- **DNS hijacking / cache poisoning** — mengalihkan korban ke server yang dikendalikan penyerang.
- **Fast-flux** — memutar DNS record secara cepat agar infrastruktur jahat selalu selangkah di depan blocklist.

Langkah pertahanan: wajibkan validasi DNSSEC, paksa klien internal menggunakan resolver Anda sendiri (bukan DNS publik sembarangan), catat volume query dan anomali, dan pertimbangkan DNS filtering untuk domain yang diketahui berbahaya.

## Firewall

### Firewall penyaring paket (packet filtering)

Memeriksa IP sumber/tujuan, protokol, dan port setiap paket terhadap aturan. Cepat dan murah, tetapi buta terhadap konteks koneksi dan isi payload.

| Jenis | Yang diperiksa | Keunggulan | Kelemahan |
|---|---|---|---|
| Packet filter (stateless) | Hanya header | Sangat cepat | Tidak ada state sesi, mudah dilewati |
| Stateful firewall | Header + state koneksi | Melacak sesi | Masih buta terhadap konten aplikasi |
| Next-generation (NGFW) | Menambah DPI, inspeksi TLS | Kesadaran aplikasi, integrasi IPS | Biaya, kompleksitas dekripsi |
| WAF | Request HTTP/HTTPS | Memblokir serangan web (SQLi, XSS) | Hanya menangani trafik web, perlu penyetelan |

### Web Application Firewall (WAF)

WAF berada di depan aplikasi web dan menerapkan aturan pada request HTTP — misalnya, menolak query string yang memuat `UNION SELECT`. Ini adalah kontrol kompensasi, bukan pengganti menulis kode yang aman: penyerang mengkode, memecah (chunk), dan mengaburkan payload untuk menguji bypass.

Contoh maksud aturan (pseudocode, bukan sintaks vendor):

```
IF http.request.uri.query CONTAINS "' OR 1=1--" THEN BLOCK
IF http.request.header.user-agent IS EMPTY THEN LOG
```

## VPN vs Proxy

| Aspek | VPN | Proxy |
|---|---|---|
| Cakupan | Mengalirkan seluruh trafik perangkat (atau satu jaringan) | Mengalirkan satu aplikasi atau request |
| Enkripsi | Biasanya full tunnel dengan kripto kuat | Bervariasi; banyak proxy hanya HTTP CONNECT biasa |
| Identitas | Mengautentikasi perangkat/pengguna | Sering anonim, tetapi operator proxy melihat semuanya |
| Penggunaan umum | Akses jarak jauh ke sumber daya perusahaan, privasi di Wi-Fi tepercaya | Penyaringan konten, caching, anonimitas sederhana |
| Kepercayaan | Anda mempercayakan metadata Anda ke penyedia VPN | Anda mempercayakan request lengkap ke proxy |

Tidak ada yang membuat Anda anonim secara default. VPN menggeser siapa yang bisa melihat trafik Anda; proxy menggeser siapa yang mengirim request atas nama Anda.

## IDS dan IPS

**IDS** (Intrusion Detection System) memantau salinan trafik jaringan dan memunculkan peringatan. **IPS** (Intrusion Prevention System) berada inline dan dapat menjatuhkan paket jahat sebelum mencapai target.

- Berbasis signature: cepat, presisi, tetapi buta terhadap serangan baru.
- Berbasis anomali/perilaku: menangkap aktivitas baru, tetapi menghasilkan false positive yang perlu disetel.
- Penempatan penting: IDS pada port SPAN hanya melihat trafik yang di-mirror kepadanya; IPS gagal terbuka atau tertutup tergantung konfigurasi saat terjadi gangguan.

Peringatan hanya berguna jika ditriase. Pertanyaan wawancara yang umum: apakah peringatan `ET EXPLOIT` berarti telah terjadi kompromi? Jawabannya: peringatan itu berarti ada yang cocok dengan sebuah pola, dan Anda tetap perlu memverifikasi konteksnya.

## Segmentasi Jaringan

Segmentasi membatasi pergerakan lateral dengan membagi jaringan menjadi zona-zona yang trafiknya terkontrol di antaranya. Jika penyerang membajak sebuah server web, segmentasi menghentikan mereka mencapai VLAN keuangan secara langsung.

Langkah praktis:

1. Inventarisasi aset dan klasifikasikan data (publik, internal, rahasia).
2. Kelompokkan berdasarkan fungsi dan sensitivitas: pengguna, server, basis data, IoT/OT, manajemen.
3. Izinkan hanya aliran yang diperlukan antar zona (default deny).
4. Lindungi jalur administratif dengan jump host atau privileged access workstation.
5. Pantau trafik east-west, bukan hanya north-south.

Microsegmentation membawa konsep ini ke tingkat beban kerja (workload), sering diterapkan oleh firewall host atau cloud security group, bukan switch fisik.

## Zero Trust: Pengantar Singkat

Zero trust menggantikan asumsi lama "percaya pada jaringan internal" dengan prinsip: **jangan pernah percaya, selalu verifikasi**. Ide intinya meliputi:

- Autentikasi dan otorisasi setiap request, tanpa memedulikan jaringan sumbernya.
- Gunakan kesehatan perangkat dan konteks (lokasi, tingkat patch) dalam keputusan akses.
- Berikan akses privilege terbatas dan berumur pendek, bukan hak istimewa permanen.
- Asumsikan sudah terjadi breach: catat semuanya, segmentasikan semuanya, siapkan penanganan (containment).

Zero trust adalah sebuah program, bukan produk. Membeli "perangkat zero trust" tanpa kematangan identitas, segmentasi, dan logging tidak akan mewujudkan zero trust.

## Skenario: Menilai Koneksi Keluar yang Mencurigakan

Anda melihat sebuah workstation melakukan query DNS ke domain yang tidak dikenal setiap 60 detik:

```
$ ps -ef | grep -i updater
$ sudo lsof -i -P -n | grep ESTABLISHED
updater  1843  user  4u  IPv4  55211  0t0  TCP 10.0.4.23:51204->203.0.113.77:443 (ESTABLISHED)
$ dig +short suspicious-example.xyz
203.0.113.77
```

Tindakan: isolasikan host dari jaringan, ambil bukti volatil, periksa telemetri EDR, dan blokir domain/IP di resolver serta perimeter selama Anda menyelidikinya. Pola berulang ditambah domain ber reputasi buruk adalah indikator kuat adanya C2 beaconing.

## Lihat juga

- [Triad CIA](cia-triad.md)
- [Dasar Kriptografi](cryptography-basics.md)
- [Keamanan Web & OWASP Top 10](web-security-owasp.md)
- [Keamanan Sistem Operasi](operating-system-security.md)
- [Jalur Belajar Keamanan Siber](../career/learning-path.md)
