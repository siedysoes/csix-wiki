# Triad CIA

Triad CIA adalah model tiga tujuan keamanan — kerahasiaan (confidentiality), integritas (integrity), dan ketersediaan (availability) — yang dipakai untuk mendefinisikan kebutuhan dan menjelaskan mengapa sebuah insiden itu penting. Hampir setiap kontrol yang Anda temui, mulai dari enkripsi disk hingga backup luar kantor, ada untuk melindungi setidaknya satu dari tiga sifat ini.

## Kerahasiaan

Kerahasiaan berarti hanya pihak yang berwenang yang boleh membaca datanya. Fokusnya adalah mencegah kebocoran — bukan hanya dari penyerang, tetapi juga dari karyawan yang tidak berwenang, pengiklan, atau siapa pun yang menyadap lalu lintas saat sedang dikirim.

Contoh nyata:

- Enkripsi data tersimpan (BitLocker, LUKS) dan data dalam perjalanan (TLS) sehingga data yang tertangkap tidak terbaca tanpa kunci.
- Kontrol akses: izin berbasis peran, hak sesuai kebutuhan (least privilege), prinsip "perlu tahu" (need to know), dan autentikasi multifaktor (MFA).
- Kebijakan klasifikasi data yang menandai catatan pelanggan sebagai terbatas dan membatasi siapa yang boleh mengekspornya.

**Bagaimana pelanggaran melanggarnya:** penyerang mencuri basis data dan mempublikasikannya. Kebocoran Equifax 2017 yang terkenal membocorkan data pribadi sekitar 147 juta orang — data diungkapkan kepada pihak yang tidak berhak melihatnya, murni kegagalan kerahasiaan.

**Trade-off:** kerahasiaan yang kuat justru bisa menurunkan ketersediaan — jika satu-satunya salinan kunci enkripsi hilang, datanya ikut hilang. Aturan need-to-know yang ketat memperlambat kolaborasi, dan enkripsi end-to-end bertentangan dengan organisasi yang ingin memindai konten untuk mendeteksi malware atau memenuhi kepatuhan.

## Integritas

Integritas berarti data akurat, lengkap, dan hanya diubah oleh pihak yang berwenang dengan cara yang sah. Ini mencakup perusakan data secara tidak sah maupun penghancurannya secara tidak sah.

Contoh nyata:

- Hash dan tanda tangan digital untuk mendeteksi manipulasi (tanda tangan pembaruan perangkat lunak, checksum SHA-256 untuk unduhan).
- Alat pemantauan integritas berkas (AIDE, Tripwire) yang memberi peringatan saat berkas sistem berubah secara tak terduga.
- Kontrol versi, log audit, batasan basis data, dan proses tinjauan perubahan yang membuat penyuntingan tak sah terlihat.

**Bagaimana pelanggaran melanggarnya:** penyerang memodifikasi catatan transaksi untuk mengalihkan pembayaran, atau menyunting berkas log untuk menghapus jejak intrusi. Ransomware juga merusak integritas — berkas yang dienkripsi menjadi rusak dan tak terpakai meskipun tidak ada teks biasa yang bocor.

**Trade-off:** kontrol integritas menambah friksi. Persetujuan empat mata (dua pemeriksa), code review, dan log audit yang tidak bisa diubah memperlambat proses rilis. Sifat tak terubah (immutability) ini juga bertentangan dengan peraturan privasi yang memberi orang hak untuk menghapus datanya.

## Ketersediaan

Ketersediaan berarti pengguna yang berwenang dapat mengakses sistem dan data saat membutuhkannya. Sistem yang sangat rahasia dan utuh tetapi tak bisa dijangkau siapa pun di saat kritis bagi bisnis tetap saja gagal.

Contoh nyata:

- Redundansi: array RAID, replika basis data, klaster failover, dan beberapa availability zone.
- Backup mengikuti aturan 3-2-1 (3 salinan, 2 media, 1 di lokasi luar) dengan uji pemulihan yang dilakukan rutin.
- Mitigasi DDoS, autoscaling, load balancing, dan patching tepat waktu untuk mencegah gangguan layanan.

**Bagaimana pelanggaran melanggarnya:** banjir DDoS membuat situs down, atau ransomware mengenkripsi semua workstation — WannaCry 2017 membuat rumah sakit tak bisa mengakses catatan pasien selama berhari-hari. Kegagalan perangkat keras tanpa backup yang sudah diuji adalah kehilangan ketersediaan tanpa adanya penyerang sama sekali.

**Trade-off:** ketersediaan tinggi butuh biaya dan kompleksitas operasional. Patching yang ketat mungkin butuh jadwal pemeliharaan, pembatasan laju (rate limiting) yang agresif bisa memblokir pengguna sah, dan replikasi sinkron antarwilayah meningkatkan ketersediaan sekaligus menambah latensi.

## Trade-off antar ketiganya

Keputusan keamanan biasanya menyeimbangkan ketiga tujuan tersebut satu sama lain.

| Keputusan | Kerahasiaan | Integritas | Ketersediaan |
|---|---|---|---|
| Enkripsi seluruh disk di laptop | Lebih kuat | Netral | Lebih lemah jika kunci hilang |
| Replika basis data hanya-baca | Netral | Lebih kuat (satu sumber tulis) | Lebih kuat (failover) tetapi proses tulis bisa tertahan |
| Backup offline terisolasi jaringan (air-gapped) | Lebih kuat (tak bisa dijangkau jaringan) | Lebih kuat (tahan manipulasi) | Lebih lemah (pemulihan butuh waktu) |
| Persetujuan wajib sebelum perubahan ke produksi | Netral | Lebih kuat | Sedikit lebih lemah (perbaikan insiden lebih lambat) |
| Rate limiting / proteksi bot | Netral | Netral | Lebih kuat bagi layanan, lebih lemah bagi pengguna berat yang sah |

Kebiasaan yang berguna: untuk setiap kontrol yang Anda rancang atau tinjau, tanyakan ketiga hal — melindungi yang mana, melemahkan yang mana, dan apakah keseimbangan itu sesuai dengan risikonya.

## Skenario

Sebuah toko online mengevaluasi tiga usulan untuk basis data pelanggannya:

1. Menyimpannya di satu PC kantor tanpa backup. Murah, tetapi kegagalan disk atau insiden ransomware apa pun bisa menghancurkan bisnis ini.
2. Menaruhnya di bucket cloud publik dengan akses terbuka ke semua orang agar tim pemasaran bisa "tinggal ambil yang mereka butuhkan". Cepat untuk dikerjakan, tetapi kerahasiaannya hilang.
3. Mengenkripsi basis data, membatasi akses ke dua peran, mencatat setiap query, mereplikasi ke wilayah kedua, serta mengambil backup setiap malam dengan uji pemulihan bulanan.

Usulan 3 adalah yang paling seimbang: kerahasiaan (enkripsi, kontrol akses), integritas (pencatatan query, jalur tulis yang dibatasi), dan ketersediaan (replikasi, backup yang sudah diuji) semuanya terpenuhi, dengan biaya yang diketahui dan diterima.

## Menguji ketiga sifat

Kontrol hanya nyata jika Anda membuktikannya.

| Sifat | Cara mengujinya |
|---|---|
| Kerahasiaan | Tinjau izin akses, coba membaca sebagai pengguna tak berwenang, pastikan enkripsi pada data tersimpan dan yang dikirim |
| Integritas | Bandingkan hash berkas dengan nilai yang diketahui benar, periksa log audit untuk perubahan tak terduga, latih pemulihan dari backup |
| Ketersediaan | Jalankan latihan failover, ukur waktu pemulihan backup penuh, uji beban layanan, simulasikan gangguan |

## Kesalahan umum

- Memperlakukan triad sebagai tiga kotak centang, bukan sebagai keseimbangan — satu kontrol bisa menguatkan satu tujuan sambil melemahkan tujuan lain.
- Menganggap kerahasiaan menjamin integritas: data terenkripsi tetap bisa diubah kecuali ia juga diautentikasi (lihat authenticated encryption).
- Menyamakan ketersediaan dengan kecepatan: sistem lambat yang akhirnya merespons pun bisa tetap gagal memenuhi kebutuhan bisnis.
- Menguji sekali lalu tidak pernah lagi: backup membusuk, izin menumpuk, dan jalur failover putus tanpa disadari.

## Model terkait

Triad CIA adalah titik awal, bukan satu-satunya model:

- **Parkerian hexad** (Donn Parker) memperluas triad dengan kepemilikan (possession), keaslian (authenticity), dan kegunaan (utility).
- **Triad DAD** (Disclosure, Alteration, Destruction — kebocoran, perubahan, penghancuran) menggambarkan apa yang dicapai penyerang — cermin ofensif dari CIA.
- **Model privasi** menambahkan pertanyaan penggunaan yang sah dan persetujuan (consent) yang tidak ditangkap oleh CIA saja.
- Framework seperti **NIST CSF** dan **ISO 27001** mengorganisasi kontrol di sekitar hasil-hasil semacam CIA.

## Lihat juga

- [Lanskap Ancaman](threat-landscape.md)
- [Dasar Kriptografi](cryptography-basics.md)
- [Keamanan Web & OWASP Top 10](web-security-owasp.md)
- [Keamanan Sistem Operasi](operating-system-security.md)
