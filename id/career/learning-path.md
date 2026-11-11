# Jalur Belajar Cybersecurity

Beralih ke dunia keamanan akan jauh lebih mudah jika Anda membangun di atas pengetahuan IT umum, bukan langsung melompat ke penggunaan tool. Halaman ini menjelaskan tahapan dari pemula total hingga siap kerja, bidang spesialisasi utama yang bisa Anda pilih setelahnya, dan di mana Anda bisa berlatih — ini adalah urutan yang harus diikuti, bukan daftar centang yang buru-buru dituntaskan.

## Ikhtisar tahapan

| # | Tahapan | Yang seharusnya bisa Anda lakukan | Perkiraan durasi (paruh waktu) |
|---|---------|-----------------------------------|-------------------------------|
| 1 | Dasar-dasar IT | Menjelaskan cara kerja hardware, sistem operasi, file dan proses; memecahkan masalah dasar | 2-3 minggu |
| 2 | Jaringan | Subnetting, model TCP/IP, DNS, DHCP, HTTP/HTTPS, port umum; membaca packet capture | 4-6 minggu |
| 3 | Sistem operasi | Bekerja di terminal Linux dan command line Windows; mengelola user, permission, service dan log | 3-4 minggu |
| 4 | Scripting | Menulis script Python kecil (plus Bash atau PowerShell) untuk mengotomatiskan tugas berulang | 3-4 minggu |
| 5 | Dasar-dasar keamanan | Menerapkan triad CIA, mengenali malware dan social engineering, menjelaskan kriptografi dan autentikasi dasar | 3-4 minggu |
| 6 | Lab praktik | Menyelesaikan room berpandu, tantangan CTF, dan mesin yang sengaja dibuat rentan | berkelanjutan |
| 7 | Spesialisasi | Mendalami satu jalur sambil mempertahankan pengetahuan kerja di jalur lainnya | terus-menerus |

Durasi di atas bersifat perkiraan; urutannya lebih penting daripada kecepatannya. Jangan lanjut ke tahap berikutnya sampai Anda bisa menjelaskan tahap saat ini tanpa catatan.

### Catatan per tahap

- **Dasar-dasar IT**: tahu apa itu CPU, disk, memori dan proses; memahami virtualisasi cukup untuk menjalankan mesin virtual; nyaman menginstal software dan membaca pesan error.
- **Jaringan**: tahap yang paling sering dilewati dan paling sering disesali. Fokus pada model TCP/IP, addressing dan subnetting, DNS, DHCP, TLS, HTTP serta port umum, lalu berlatih dengan analis paket seperti Wireshark.
- **Sistem operasi**: Linux muncul di sebagian besar tool keamanan, Windows muncul di sebagian besar audit. Pelajari command line, permission file, user dan grup, service, serta di mana log tersimpan di masing-masing sistem.
- **Scripting**: Python biasanya menjadi pilihan pertama karena library standarnya sudah mencakup penanganan file, parsing dan HTTP. Otomatiskan tugas apa pun yang sudah pernah Anda kerjakan manual sebanyak dua kali.
- **Dasar-dasar keamanan**: pelajari konsepnya sebelum tool-nya — triad CIA, threat actor, kategori malware, social engineering, dasar kriptografi, password dan autentikasi, serta OWASP Top 10 untuk web.
- **Lab praktik**: teori tanpa latihan tidak akan bertahan dalam wawancara. Gunakan target yang sengaja dibuat rentan hanya di lingkungan lab yang Anda izinkan untuk diuji.

## Peta spesialisasi

| Jalur | Yang mereka lakukan sehari-hari | Peran pertama yang umum |
|-------|--------------------------------|--------------------------|
| Blue team / SOC | Memantau alert, mengklasifikasi insiden, menyesuaikan deteksi serta menulis laporan atau rule deteksi, sering kali dengan SIEM | Analis SOC / analis keamanan junior |
| Red team / pentest | Menguji sistem dengan izin tertulis, menemukan dan membuktikan kerentanan, menulis temuan dan rekomendasi | Junior penetration tester / security consultant |
| Application security | Meninjau kode, membuat threat model untuk fitur, dan menjalankan pengujian keamanan dalam siklus pengembangan | AppSec analyst / junior security engineer |
| Cloud security | Mengamankan akun cloud, identitas dan akses, penyimpanan serta jaringan; memburu miskonfigurasi | Cloud security analyst (biasanya setelah peran sysadmin atau cloud admin) |
| GRC | Kebijakan, penilaian risiko, audit, dan kepatuhan terhadap framework seperti NIST dan ISO 27001 | GRC / risk / compliance analyst |

Anda tidak harus memilih di hari pertama. Jalankan satu proyek pemula dari dua atau tiga jalur, perhatikan pekerjaan mana yang dengan senang hati ingin Anda ulangi, lalu berkomitmen — tetapi pada bulan ke-6 studi Anda, Anda seharusnya sudah condong ke satu arah, karena iklan lowongan kerja menghargai fokus.

## Sumber daya per kategori

### Dokumentasi dan referensi resmi

- NIST Computer Security Resource Center — standar dan pedoman publik yang digunakan di seluruh dunia
- CISA — advisory, alert, dan pedoman defensif gratis
- OWASP — Testing Guide, Cheat Sheets, dan Top 10; wajib untuk pekerjaan web dan AppSec
- MITRE ATT&CK — basis pengetahuan bersama untuk mendeskripsikan perilaku penyerang
- Dokumentasi vendor yang benar-benar akan Anda gunakan: Microsoft Learn, dokumentasi distribusi Linux Anda, tutorial resmi Python

### Platform lab

- **TryHackMe** — room berpandu berbasis browser yang mengajarkan langkah demi langkah; titik masuk paling ramah bagi pemula
- **Hack The Box** — mesin dan tantangan yang mengasumsikan lebih banyak kemandirian; bagus ketika room berpandu mulai terasa mudah
- **OverTheWire** — wargame gratis yang melatih Linux, shell, dan dasar-dasar keamanan tanpa perlu instalasi

### Capture the flag (CTF)

- **picoCTF** — ramah pemula dan sering diadakan sebagai acara sekolah atau komunitas; CTF pertama yang baik
- **CTFtime** — kalender dan papan skor acara CTF publik di seluruh dunia, plus arsip write-up yang dipublikasikan
- Acara klub lokal dan daring — banyak komunitas mengadakan acara latihan yang bisa Anda ikuti tanpa bepergian

Pilih satu platform lab dan satu set referensi, lalu selesaikan sesuatu di sana sebelum membuka sepuluh tab browser. Satu room yang selesai beserta ringkasan tertulis mengajarkan lebih banyak daripada folder bookmark berisi setengah jalan, dan nama course sering berubah — berpeganglah pada tautan platform dan dokumentasi di atas daripada course pihak ketiga tertentu.

## Lihat juga

- [Dasar-Dasar Keamanan Jaringan](../fundamentals/network-basics.md)
- [Keamanan Sistem Operasi](../fundamentals/operating-system-security.md)
- [Keamanan Web & OWASP Top 10](../fundamentals/web-security-owasp.md)
- [Panduan Sertifikasi Keamanan](certifications.md)
