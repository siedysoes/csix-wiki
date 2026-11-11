# Peta Belajar: 3 / 6 / 12 Bulan

Halaman ini mengubah jalur belajar menjadi tiga rencana konkret, tergantung pada berapa banyak waktu yang Anda miliki dan apakah Anda sudah bekerja di bidang IT. Setiap tabel memuat tujuan bulanan, kebiasaan mingguan, serta milestone atau checkpoint yang harus Anda lalui sebelum lanjut — perlakukan checkpoint sebagai gerbang, bukan saran.

## Memilih lini waktu

| Lini waktu | Paling cocok untuk | Yang realistis bisa dihasilkan |
|----------|----------|------------------------------------|
| 3 bulan | Orang dengan pengalaman IT atau developer yang sudah ada, belajar penuh waktu | Fondasi yang solid plus persiapan sertifikasi entry; pekerjaan pertama mungkin bisa, tetapi persaingannya akan ketat |
| 6 bulan | Pindah karier yang belajar paruh waktu dengan jadwal konsisten | Fondasi, satu sertifikasi entry, dan portofolio kecil |
| 12 bulan | Pemula atau yang menggabungkan belajar dengan pekerjaan penuh waktu | Fondasi, sertifikasi, uji spesialisasi, portofolio, dan kesiapan wawancara |

## Rencana 3 bulan (intensif)

| Bulan | Tujuan | Kebiasaan mingguan | Milestone / checkpoint |
|-------|-------|---------------|-------------------------|
| 1 | Inti dasar IT dan jaringan: model TCP/IP, subnetting, DNS, DHCP, HTTP/TLS, port umum | 8-10 jam belajar; satu capture Wireshark dianalisis per minggu; tinjauan terminologi harian | Menelusuri satu query DNS dari awal hingga akhir dari ingatan; menyelesaikan latihan subnetting tanpa kalkulator |
| 2 | Dasar-dasar Linux dan Windows plus dasar scripting Python | Sesi terminal harian; satu script otomasi kecil per minggu (pencatat ringkasan log, pengecek port) | Mengadministrasi satu VM Linux sepenuhnya dari CLI; menjalankan script yang membaca file log dan mencetak ringkasan |
| 3 | Dasar-dasar keamanan, lab berpandu, persiapan sertifikasi entry | 3-5 room lab berpandu atau setara; write-up singkat setelah setiap tantangan yang selesai | Menyelesaikan satu lab pemula dari awal hingga akhir tanpa bantuan; memperoleh skor konsisten pada set latihan soal lengkap untuk sertifikasi entry pilihan Anda |

## Rencana 6 bulan (seimbang)

| Bulan | Tujuan | Kebiasaan mingguan | Milestone / checkpoint |
|-------|-------|---------------|-------------------------|
| 1 | Dasar-dasar IT: hardware, file, proses, virtualisasi, pemecahan masalah | 5-7 jam belajar; membangun satu VM dan mendokumentasikan konfigurasinya | Menjelaskan setiap komponen komputer Anda sendiri dan memulihkan VM yang sengaja dibuat rusak dari snapshot |
| 2 | Mendalami jaringan: addressing, routing, DNS, DHCP, TLS, HTTP, port | Analisis paket dua kali seminggu; tinjauan kartu (flashcard) mingguan | Menangkap dan menjelaskan satu sesi HTTPS nyata: DNS, handshake TCP, negosiasi TLS, permintaan HTTP |
| 3 | Sistem operasi: Linux CLI, permission, service, log; padanan di Windows | Selang-seling sesi Linux dan Windows; satu tugas hardening per minggu | Menyiapkan user, permission, dan satu service di Linux, lalu menemukan lognya di kedua platform |
| 4 | Scripting dan otomasi dengan Python (plus pengenalan Bash atau PowerShell) | Satu script per minggu yang menghapus satu langkah manual dari alur kerja Anda sendiri | Mempublikasikan script yang berfungsi dengan README yang bisa dijalankan orang asing tanpa bertanya kepada Anda |
| 5 | Dasar-dasar keamanan: triad CIA, ancaman, malware, social engineering, kriptografi, autentikasi, OWASP Top 10 | Latihan skenario; mulai jalur platform lab untuk arah pilihan Anda | Menjelaskan setiap kategori OWASP Top 10 dengan satu contoh konkret dan satu mitigasi |
| 6 | Lab berpandu, keluaran portofolio pertama, uji spesialisasi | 4-6 lab atau tantangan CTF; dua write-up terpublikasi | Menyelesaikan rangkaian lab pemula hingga menengah dan memilih spesialisasi pertama Anda |

## Rencana 12 bulan (menyeluruh)

| Bulan | Fokus | Kebiasaan mingguan | Milestone / checkpoint |
|-------|-------|---------------|-------------------------|
| 1-2 | Dasar-dasar IT dan jaringan | 4-6 jam per minggu; satu capture paket dianalisis per minggu | Subnetting, TCP/IP, dan DNS dijelaskan dari ingatan; satu file catatan jaringan tertulis yang bisa Anda tinjau ulang |
| 3-4 | Sistem operasi: administrasi Linux dan Windows | Selang-seling sesi OS; latihan berburu log mingguan | Mengadministrasi kedua platform dari command line dan menemukan log yang relevan untuk peristiwa tertentu |
| 5-6 | Scripting dan otomasi | Satu proyek Python setiap dua minggu; membaca kode orang lain setiap minggu | Tiga tool kecil di satu repositori GitHub dengan README |
| 7-8 | Dasar-dasar keamanan plus sertifikasi entry | Soal skenario per minggu; sesi lab per minggu | Lulus satu ujian latihan lengkap untuk sertifikasi entry pilihan Anda, lalu jadwalkan ujian yang asli |
| 9-10 | Lab praktik dan uji spesialisasi | 2-3 lab atau tantangan CTF per minggu; satu write-up per mesin yang selesai | Menyelesaikan rangkaian yang koheren di satu jalur (blue, red, AppSec, cloud, atau GRC) dan menjelaskan mengapa Anda memilihnya |
| 11 | Pendalaman portofolio plus kredensial menengah atau proyek serius | Publikasi mingguan; tinjau write-up Anda sendiri untuk mencari celah | Portofolio dengan 4+ write-up, satu proyek GitHub, dan diagram lab; sertifikasi menengah dicoba jika waktu memungkinkan |
| 12 | Persiapan wawancara dan lamaran | Dua wawancara simulasi atau latihan teknis per minggu; melamar secara rutin | Menjawab pertanyaan jaringan dan triage tanpa diminta; memperkenalkan portofolio Anda kepada peninjau dalam 10 menit |

## Membangun home lab dengan aman dan legal

- **Isolasikan**: gunakan hypervisor (VirtualBox, VMware Workstation Player, atau sejenisnya) dengan jaringan host-only atau internal. Jangan pernah menjembatani VM yang rentan ke jaringan produksi Anda, dan jangan port-forward ke internet.
- **Gunakan image terpercaya**: ISO OS resmi, image yang sengaja dibuat rentan dari proyek bereputasi, atau mesin virtual yang disediakan platform lab. Verifikasi unduhan terhadap checksum yang dipublikasikan jika tersedia.
- **Snapshot dulu**: ambil snapshot bersih sebelum setiap latihan dan kembalikan setelahnya, sehingga eksperimen yang gagal tidak merugikan Anda.
- **Uji hanya milik Anda atau yang Anda izinkan untuk diserang**: itu termasuk sistem pemberi kerja Anda — cakupan tertulis itu penting. Jangan memindai atau menyerang rentang IP publik "untuk latihan"; gunakan rentang VPN platform lab, yang secara eksplisit berada dalam cakupan.
- **Jaga kelemahan yang disengaja tetap terkontrol**: image yang rentan memang tidak aman secara desain, jadi jalankan secara offline atau di jaringan terisolasi dan jangan pernah menggunakan kredensialnya lagi di tempat lain.

## Portofolio dan bukti karya

- **Write-up lab dan CTF**: metodologi, perintah yang digunakan, apa yang gagal lebih dulu, apa yang Anda pelajari. Hindari spoiler untuk tantangan yang masih berjalan, dan jangan pernah mempublikasikan materi yang melanggar aturan platform.
- **Proyek GitHub**: tool kecil yang benar-benar berjalan — pencatat ringkasan log, pengecek security header, pengelola wordlist — masing-masing dengan README, langkah instalasi, dan contoh keluaran.
- **Konten deteksi**: untuk arah blue team, publikasikan rule (misalnya deteksi bergaya Sigma) yang diturunkan dari perilaku yang Anda amati di lab, dengan penjelasan singkat tentang sumber lognya.
- **Dokumentasi lab**: diagram jaringan home lab Anda plus checklist hardening yang Anda terapkan, yang sekaligus bisa menjadi bahan wawancara.
- **Seri blog singkat**: jelaskan satu konsep yang harus Anda perjuangkan; menjelaskan adalah cara Anda mengetahui apa yang belum Anda pahami.

## Persiapan wawancara

- **Perilaku**: siapkan jawaban berformat STAR untuk alasan memilih keamanan, satu proyek yang gagal, satu konflik di tim, cara Anda terus belajar, dan cara Anda menghadapi ambiguitas atau tenggat yang ketat. Setiap poin di resume Anda harus bisa dijelaskan dalam dua menit.
- **Teknis (defensif)**: uraikan proses triase terhadap satu sampel alert — log dan sistem mana yang Anda periksa lebih dulu, bagaimana Anda memisahkan indikator dari false positive, dan bagaimana Anda akan mendokumentasikan insidennya. Segarkan kembali DNS, handshake TCP, TLS, dan sumber log yang umum.
- **Teknis (ofensif)**: deskripsikan metodologi Anda untuk satu engagement host atau web, urutan enumerasi Anda, apa yang Anda lakukan saat mandek, dan bagaimana Anda memutuskan apa yang masuk ke laporan.
- **Praktis**: siapkan untuk tugas screen-share seperti membaca cuplikan log atau menulis script singkat. Berlatihlah menceritakan proses berpikir Anda sambil bekerja.
- **Pertanyaan untuk mereka**: tanyakan tentang tooling tim, proses on-call atau eskalasi, dan bagaimana keberhasilan diukur dalam enam bulan pertama.

## Jebakan yang umum

| Jebakan | Tampilannya | Solusi |
|---------|--------------------|-----|
| Tutorial hell | Course tanpa henti, tidak ada yang dibangun sendiri | Tutup tutorial dan bangun ulang hasil yang sama dari nol, lalu dokumentasikan |
| Mengumpulkan sertifikasi tanpa keterampilan | Tumpukan voucher tetapi tidak bisa menjelaskan subnetting atau membaca log | Satu sertifikasi dalam satu waktu, keterampilan dulu, ujian kemudian |
| Melewatkan dasar-dasar jaringan | Tidak bisa menginterpretasikan packet capture atau menjelaskan resolusi DNS | Kembali ke tahap jaringan sebelum membeli apa pun lagi |
| Hanya teori, tanpa lab | Definisi benar, nol jam praktik | Jadwalkan waktu lab di kalender dengan bobot yang sama seperti membaca |
| Pekerjaan tanpa dokumentasi | Tidak ada yang konkret untuk ditunjukkan dalam wawancara | Publikasikan satu write-up atau repo setiap kali lab selesai |
| Pengujian tanpa izin | Memindai jaringan atau host di luar cakupan tertulis | Berlatih hanya di lab Anda atau di lingkungan platform yang berada dalam cakupan |

## Lihat juga

- [Jalur Belajar Cybersecurity](learning-path.md)
- [Panduan Sertifikasi Keamanan](certifications.md)
- [Dasar-Dasar Keamanan Jaringan](../fundamentals/network-basics.md)
- [Keamanan Sistem Operasi](../fundamentals/operating-system-security.md)
