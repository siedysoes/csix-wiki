# Membangun Home Lab

Home lab adalah lingkungan privat dan terisolasi — mesin virtual, jaringan virtual, dan perangkat keamanan — tempat Anda berlatih menyerang dan membela sistem yang Anda miliki sendiri. Home lab mengubah bacaan pasif menjadi keterampilan praktis, dan memberi Anda tempat aman untuk merusak sesuatu, memulihkannya, dan menjelaskan apa yang terjadi.

## Etika dan legalitas lebih dulu

Satu aturan mengatur segala sesuatu di halaman ini: **hanya uji sistem yang Anda miliki atau yang memiliki izin tertulis, eksplisit untuk diuji.** Memindai, mengeksploitasi, atau bahkan sekadar terhubung ke sistem tanpa izin merupakan tindak pidana di sebagian besar yurisdiksi, dan hukumnya berbeda antar negara — "Saya cuma berlatih" bukan pembelaan.

- Jaga selalu lalu lintas lab tetap di dalam ruang alamat milik lab. Rentang `192.0.2.0/24`, `198.51.100.0/24`, dan `203.0.113.0/24` dipesan untuk dokumentasi dan tidak pernah dirutekan di internet.
- Berikan setiap target rentan dan alat serang sebuah jaringan terisolasi yang tidak dapat menjangkau perangkat di rumah Anda, tempat kerja, atau internet.
- Jangan pernah menjalankan exploit terhadap mesin pihak ketiga "untuk melihat apakah berhasil", dan jangan pernah mengunggah kredensial hasil tangkapan atau malware ke layanan publik.
- Jika Anda menggunakan platform latihan (TryHackMe, Hack The Box, OverTheWire), ruang lingkup platform itulah yang menentukan apa yang berwenang Anda lakukan — tetap di dalamnya.
- Untuk lab kelompok, tulis ruang lingkup, teknik yang diizinkan, dan kontak darurat sebelum siapa pun memulai.

Jika Anda ragu apakah sesuatu sudah berwenang, hentikan dan tanyakan kepada pemiliknya secara tertulis.

## Opsi virtualisasi

| Opsi | Berjalan di | Cocok untuk | Keterbatasan |
|---|---|---|---|
| VirtualBox | Windows, macOS, Linux | VM lengkap gratis, ramah pemula | I/O disk lebih lambat daripada hypervisor berbayar |
| VMware Workstation Pro / Fusion | Windows, macOS, Linux | Performa dan snapshot lebih baik (gratis untuk penggunaan pribadi) | Lisensi untuk penggunaan komersial |
| WSL 2 | Windows | Perangkat Linux (nmap, tcpdump, Python) tanpa VM penuh | Bukan batas keamanan; tidak bisa menjadi tuan rumah target guest |
| Proxmox VE / ESXi | Host bare-metal | Lab jangka panjang di PC cadangan | Butuh perangkat khusus |
| Docker | Host apa pun dengan runtime kontainer | Target web ringan seperti DVWA atau Juice Shop | Kontainer berbagi kernel host; isolasi lebih lemah daripada VM |

Aturan praktis: gunakan VM untuk target yang akan Anda serang dan eksploitasi, dan gunakan WSL atau kontainer untuk perangkat serta target web sederhana.

## Jaringan terisolasi

Jangan pernah menjembatani lab VM ke LAN asli Anda. Buatlah jaringan privat sebagai gantinya:

- **Host-only network** — VM dan host dapat saling berbicara, tidak ada yang lain. Di VirtualBox ini membuat adapter seperti `192.168.56.1`; ideal ketika VM penyerang Anda harus menjangkau VM target tetapi keduanya tidak membutuhkan internet.
- **Internal network** — hanya VM-ke-VM, tanpa akses host. Gunakan untuk segmen yang ingin Anda putus sepenuhnya.
- **NAT** — untuk keluar ke internet demi pengunduhan. Gunakan hanya pada VM penyerang ketika Anda membutuhkan pembaruan paket, lalu putuskan kembali.
- **Tanpa adapter (air gap)** — untuk analisis malware dan penangkapan lalu lintas berbahaya.

Susunan yang umum: VM penyerang di `192.168.56.0/24` (host-only) plus adapter kedua pada NAT untuk pembaruan; VM target hanya mendapatkan adapter host-only. Verifikasi dengan `ip addr` di Linux atau `ipconfig` di Windows sebelum menjalankan apa pun.

## Strategi snapshot

Snapshot memungkinkan Anda mengembalikan mesin ke keadaan yang diketahui dalam hitungan detik, dan itulah yang membuat latihan berulang menjadi mungkin.

1. Instal OS guest, terapkan pembaruan, instal perangkat dasar — lalu ambil snapshot bernama `baseline` dan jangan pernah mengeditnya lagi.
2. Ambil snapshot bernama sebelum setiap langkah berisiko: `pre-exploit`, `pre-patch`, `configured`.
3. Kembali ke `baseline` setelah setiap latihan agar run berikutnya dimulai dari keadaan yang sama. Mengembalikan snapshot membuang semua yang direkam setelah snapshot itu, jadi snapshot antara hanya untuk penyimpangan singkat.
4. Simpan dua atau tiga snapshot per VM. Snapshot tersimpan di disk host dan bertambah saat VM menulis; rantai yang panjang mudah rusak.

Snapshot bukan cadangan. Simpan ekspor sungguhan untuk apa pun yang tidak bisa Anda bangun ulang dari instalasi bersih.

## Target yang sengaja dibuat rentan

Proyek-proyek ini memang ada untuk dihancurkan — instal hanya di dalam jaringan lab.

| Target | Yang diajarkan | Catatan |
|---|---|---|
| DVWA (Damn Vulnerable Web Application) | SQL injection, XSS, CSRF, command injection | Aplikasi PHP dengan tingkat kesulitan dari low hingga impossible |
| OWASP Juice Shop | OWASP Top 10 modern, keamanan API, blind XSS | Node.js; sering dijalankan sebagai `docker run -d -p 3000:3000 bkimminich/juice-shop` |
| Metasploitable 2 / 3 | Layanan Linux yang lemah secara sengaja, target untuk framework exploit | Untuk lab saja; jangan pernah diekspos ke jaringan mana pun yang tidak Anda miliki |
| Image VulnHub | VM rentan lengkap dengan tujuan bergaya capture-the-flag | Unduh dan impor ke hypervisor Anda |
| OWASP WebGoat | Pelajaran terpandu tentang injeksi, kontrol akses, autentikasi | Berjalan dari Docker atau JVM lokal |

Unduh hanya dari halaman proyek resmi atau organisasi GitHub/Docker Hub mereka; "VM hacking" acak dari situs tepercaya rendah adalah trik penyebaran malware yang umum.

## Penangkapan paket dengan Wireshark

Wireshark menunjukkan persis apa yang dilakukan perangkat Anda di jalur kabel — cara tercepat memahami handshake TCP, DNS, dan HTTP.

1. Pilih antarmuka yang benar: `Capture > Options`, lalu pilih adapter host-only (VirtualBox: `VirtualBox Host-Only Network`). Daftar lalu lintas langsung membantu jika Anda ragu.
2. Mulai penangkapan, lalu jalankan scan atau telusuri target.
3. Berhenti, lalu saring:
   - `tcp.port == 80` — lalu lintas HTTP
   - `ip.addr == 192.168.56.10` — satu host
   - `dns` — pencarian nama
   - `tcp.flags.syn == 1 and tcp.flags.ack == 0` — probe scan (paket SYN)
4. Klik kanan pada paket, lalu Follow > TCP Stream untuk merangkai ulang percakapan.

Untuk host tanpa antarmuka grafis, `tcpdump -i any -w lab.pcap` menulis berkas yang nanti bisa dibuka Wireshark. Ketika koneksi gagal, tangkap dari kedua ujung (penyerang dan target) — perbedaannya biasanya mengungkap penyebabnya.

## Penggunaan nmap dasar

`nmap` adalah pemindai jaringan standar. Jalankan hanya terhadap rentang lab.

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

`-sV` mem-fingerprint layanan, `-sC` menjalankan skrip default, `-p-` memindai seluruh 65535 port TCP. Kecocokan `--script vuln` adalah petunjuk, bukan bukti — verifikasi secara manual. Simpan hasil dengan `-oA scan1` untuk mendapatkan berkas `.nmap`, `.xml`, dan `.gnmap` sekaligus.

## Alternatif lab berdaya rendah

- **Satu VM penyerang plus kontainer**: satu VM Kali atau Parrot (2 vCPU, 4 GB RAM) dengan target Docker menjaga total konsumsi di bawah 6 GB.
- **WSL 2 di Windows**: jalankan nmap, Python, dan tcpdump untuk berlatih perangkat tanpa VM target apa pun.
- **Lab berbasis browser**: TryHackMe dan Hack The Box menjalankan target untuk Anda — nol sumber daya lokal dan selalu berwenang.
- **Guest minimal**: image Alpine Linux atau Debian netinstall untuk target, ratusan megabyte alih-alih instalasi desktop multi-gigabyte.
- **Bangun ulang dari `baseline` alih-alih menginstal ulang**: mengembalikan snapshot hanya butuh detik dan tanpa ruang disk.

## Tabel komponen lab

| Komponen | Tujuan | Pilihan umum |
|---|---|---|
| Hypervisor | Menjalankan VM | VirtualBox, VMware Workstation Pro/Fusion, Proxmox VE |
| Mesin penyerang | Pemindaian, eksploitasi, analisis | Kali Linux, Parrot OS, atau Debian polos dengan perangkat ditambahkan |
| Mesin target | Sistem yang Anda latih | DVWA, Juice Shop, VM bergaya Metasploitable, image VulnHub |
| Jaringan | Isolasi dari dunia nyata | Jaringan virtual host-only dan internal |
| Penangkapan dan analisis | Mengamati lalu lintas | Wireshark di host atau VM penyerang, tcpdump di guest tanpa antarmuka grafis |
| Dokumentasi | Memanfaatkan kembali dan menjelaskan pekerjaan Anda | Catatan lab: IP, kredensial, perintah, hasil |
| Snapshot | Reset cepat dan berulang | Snapshot bernama, ekspor berkala VM yang bersih |

## Rencana penyusunan akhir pekan pertama

**Sabtu pagi (2-3 jam)**

1. Instal hypervisor dan VM penyerang Linux dari image resmi; verifikasi checksum yang dipublikasikan.
2. Instal guest additions, set IP lab statis, ambil snapshot `baseline`.

**Sabtu sore (2-3 jam)**

3. Buat host-only network `192.168.56.0/24`, pasang hanya VM penyerang, dan pastikan VM tersebut mencapai host tetapi tidak ke internet.
4. Instal Wireshark, lakukan penangkapan di antarmuka host-only, dan verifikasi bahwa Anda bisa melihat lalu lintas Anda sendiri.

**Minggu pagi (2-3 jam)**

5. Tambahkan satu VM target (DVWA, atau Juice Shop di Docker) dengan hanya adapter host-only.
6. Jalankan empat perintah `nmap` di atas, simpan keluarannya, dan cocokkan port terbuka dengan hasil tangkapan Wireshark.

**Minggu sore (1-2 jam)**

7. Rusak satu hal secara sengaja (ubah konfigurasi, restart layanan), kembali ke `baseline`, dan pastikan reset berfungsi.
8. Tulis ringkasan satu halaman: topologi, IP, perintah yang dijalankan, hasil yang diamati. Halaman itulah README lab Anda sekaligus bukti untuk wawancara kerja.

Lab yang bisa Anda bangun ulang dalam satu jam dari catatan dan snapshot lebih berharga daripada lab yang Anda rakit sendiri selama akhir pekan tetapi tidak bisa direproduksi.

## Lihat juga

- [Dasar Keamanan Jaringan](../fundamentals/network-basics.md)
- [Keamanan Sistem Operasi](../fundamentals/operating-system-security.md)
- [Keamanan Web & OWASP Top 10](../fundamentals/web-security-owasp.md)
- [Jalur Belajar Cybersecurity](../career/learning-path.md)
