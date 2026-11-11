# Lanskap Ancaman

Lanskap ancaman menggambarkan siapa yang menyerang, mengapa mereka menyerang, dan bagaimana cara mereka masuk. Memahaminya membuat Anda bisa memprioritaskan pertahanan: kontrol yang menghentikan remaja yang iseng bukanlah kontrol yang sama dengan yang menghentikan kelompok berdana dan sabar yang mencuri rahasia dagang.

## Aktor ancaman

| Aktor | Motivasi khas | Kemampuan dan sumber daya | Contoh perilaku |
|---|---|---|---|
| Script kiddie | Ketenaran, rasa penasaran, kesenangan | Kemampuan rendah; memakai alat jadi dan tutorial | Menjalankan alat DDoS hasil unduhan atau meretas tampilan situs dengan exploit kit publik |
| Hacktivisme | Ideologi, protes, publisitas | Rendah sampai sedang; bekerja dalam kelompok, kuat di propaganda | Merusak tampilan situs, membocorkan dokumen untuk menyampaikan politis |
| Cybercriminal | Uang | Sedang sampai tinggi; terorganisasi, memakai phishing, ransomware, afiliasi | Pemerasan ransomware, carding, business email compromise (BEC) |
| Insider | Uang, balas dendam, kelalaian | Bervariasi; sudah punya akses sah | Menyalin daftar pelanggan, salah konfigurasi server, mengklik tautan |
| Advanced Persistent Threat (APT) | Pengintaian atau sabotase jangka panjang | Tinggi; sabar, alat khusus, menyasar organisasi tertentu | Berbulan-bulan akses diam-diam, teknik living-off-the-land, ekstraksi yang menyasar |
| Negara (nation-state) | Keunggulan geopolitik, intelijen, sabotase | Sangat tinggi; payung hukum, sumber daya sangat besar, exploit zero-day | Mencuri riset pertahanan, mengganggu infrastruktur kritis |

Catatan:

- **Insider** paling sulit dideteksi karena aktivitasnya terlihat sah. Kategori ini mencakup aktor jahat maupun sekadar ceroboh.
- **APT** adalah label untuk perilaku (persisten, terarah, canggih), bukan sinonim dari "pemerintah". Sejumlah kelompok kriminal menunjukkan ciri mirip APT.
- Operasi **negara** terkadang beririsan dengan APT; pembedanya adalah siapa yang berada di balik kelompok tersebut.

## Motif

Empat motif menjelaskan sebagian besar serangan:

1. **Uang** — ransomware, penipuan, menjual akses, cryptomining, pemerasan. Motif utama bagi kelompok kriminal.
2. **Pengintaian (espionage)** — mencuri rahasia: data militer, komersial, diplomatik, atau riset. Dominan bagi negara dan sebagian APT.
3. **Sabotase** — menghancurkan atau mengganggu operasi, dari menghapus isi disk hingga menghentikan lini produksi. Terkadang dipakai sebagai alat tawar (bayar kami dan pabrik kembali berjalan).
4. **Ideologi** — hacktivism, protes, atau keluhan pribadi. Biasanya mencari publisitas, bukan pembayaran langsung.

Motif sekunder meliputi ketenaran, balas dendam, pemerasan, dan sekadar membuat kekacauan. Kru ransomware dimotivasi uang, tetapi sering memakai dampak ala sabotase (wiper yang menghancurkan data) sebagai tekanan.

## Vektor serangan

**Vektor serangan (attack vector)** adalah jalur atau metode spesifik yang dipakai penyerang untuk mencapai target.

- **Phishing** — email, tautan, atau lampiran berbahaya yang menipu pengguna untuk menjalankan kode atau menyerahkan kredensial. Masih metode akses awal yang paling umum.
- **Drive-by download** — mengunjungi situs yang dikompromikan atau berbahaya yang diam-diam mengeksploitasi kerentanan browser atau plugin, terkadang tanpa klik apa pun selain memuat halaman.
- **Supply chain** — mengkompromikan dependensi tepercaya, server pembaruan, atau vendor. Karena muatannya datang dalam keadaan ditandatangani atau diharapkan, ia lolos dari banyak filter. Kompromi SolarWinds Orion (2020) menyuntikkan backdoor ke pembaruan perangkat lunak yang didistribusikan ke ribuan organisasi.
- **Fisik** — menumpang masuk (tailgating) ke kantor, mencuri laptop, mencolokkan perangkat USB nakal, atau memanipulasi perangkat keras.
- **Salah konfigurasi cloud** — bucket penyimpanan yang bisa dibaca publik, peran IAM yang terlalu luas, layanan menghadap internet tanpa patch. Tidak butuh keahlian eksploitasi, cukup penemuan.

## Permukaan serangan vs vektor serangan

Dua istilah ini sering tertukar:

- **Permukaan serangan (attack surface)** adalah segala sesuatu yang *bisa* dicoba penyerang: setiap port terbuka, endpoint, formulir web, API, pengguna, laptop, dan integrasi pihak ketiga.
- **Vektor serangan** adalah jalur yang *benar-benar* mereka pakai.

Contoh: sebuah toko online mengekspos port 22 (SSH), 443 (HTTPS), formulir login pelanggan, panel admin, API pembayaran, dan widget obrolan pihak ketiga. Daftar itulah keseluruhan permukaan serangan. Penyerang yang mengirim email berisi faktur palsu ke seorang akuntan dan mencuri kredensial VPN-nya telah memilih phishing sebagai vektor serangan — panel admin berada di permukaan tersebut tetapi bukan jalur yang dipakai.

Pelajaran praktisnya: perkecil permukaan (tutup port yang tak dipakai, nonaktifkan fitur yang tak dipakai), dan pantau vektor yang benar-benar dipakai penyerang (email, aplikasi web yang terbuka, kredensial).

## Memetakan serangan: kill chain dan MITRE ATT&CK

Dua framework membantu Anda memahami tahapan sebuah serangan.

### Cyber Kill Chain

Dikembangkan oleh Lockheed Martin, framework ini memodelkan intrusi terarah sebagai tujuh tahap berurutan. Membela salah satu tahap awal bisa menghentikan rantainya.

| # | Tahap | Contoh aktivitas | Contoh pertahanan |
|---|---|---|---|
| 1 | Reconnaissance | Memindai jaringan, meneliti karyawan | Minimalkan data publik, pantau layanan yang terpapar |
| 2 | Weaponization | Mengemas exploit bersama payload | Patching aplikasi, allow-listing |
| 3 | Delivery | Mengirim email phishing | Filter email, kesadaran pengguna |
| 4 | Exploitation | Memicu kerentanan | Manajemen patch, sandboxing |
| 5 | Installation | Menurunkan backdoor, kunci persistence | Deteksi endpoint, pemantauan integritas |
| 6 | Command and control | Beaconing ke server jarak jauh | Egress filtering, pemantauan DNS |
| 7 | Actions on objectives | Mencuri atau mengenkripsi data | Data loss prevention, backup, segmentasi |

### MITRE ATT&CK

ATT&CK adalah basis pengetahuan publik tentang perilaku penyerang nyata, disusun berdasarkan **taktik** (tujuan penyerang) dan **teknik** (cara mereka mencapainya). Kategori taktik utama meliputi reconnaissance, resource development, initial access, execution, persistence, privilege escalation, defense evasion, credential access, discovery, lateral movement, collection, command and control, exfiltration, dan impact.

Jika kill chain adalah cerita linier, ATT&CK adalah katalog: para defender memakainya untuk memetakan peristiwa yang terdeteksi ke ID teknik (misalnya T1566 untuk phishing), menemukan celah dalam deteksi, dan berkomunikasi secara tepat tentang ancaman. Kelompok seperti APT28 punya entri yang mencantumkan teknik-teknik spesifik yang pernah terobservasi mereka gunakan.

## Skenario

Sebuah perusahaan manufaktur menengah dikompromikan. Telusuri kill chain-nya:

1. **Reconnaissance:** penyerang menjelajahi halaman karier dan mengetahui perusahaan itu memakai perangkat VPN tertentu.
2. **Delivery:** seorang karyawan HR membuka PDF lamaran pekerjaan yang menjalankan exploit terhadap komponen yang belum dipatch.
3. **Installation dan C2:** sebuah implant kecil mengirim beacon ke host cloud sah setiap jam, menyamar di tengah lalu lintas normal.
4. **Actions on objectives:** setelah berminggu-minggu discovery dan lateral movement, penyerang mencapai server berkas dan mengekstrak gambar rekayasa.

Setiap tahap meninggalkan sinyal — PDF yang tak biasa, koneksi keluar yang aneh, akses berkas yang tidak wajar — tetapi tidak ada satu alarm pun yang cukup. Pemetaan ATT&CK akan menempatkan semuanya pada T1566 (phishing), T1071 (application layer protocol C2), dan T1005 (data from local system), yang mengungkap bahwa deteksi sudah ada untuk tahap delivery tetapi belum untuk command and control maupun exfiltration.

## Memprioritaskan pertahanan

Anda tidak bisa mempertahankan segalanya sekaligus, jadi susun urutan pekerjaannya:

1. **Tutup dulu vektor bervolume tinggi:** filter email dan MFA terhadap phishing, patching sistem yang menghadap internet, dan mengganti kredensial bawaan.
2. **Asumsikan akses awal akan terjadi** suatu saat: berinvestasi pada deteksi untuk persistence, credential access, dan lateral movement — tahap-tahap tengah dari ATT&CK.
3. **Sesuaikan upaya dengan aktor yang masuk akal.** Bisnis kecil sebagian besar menghadapi cybercriminal dan opportunist, sehingga backup serta MFA lebih bernilai daripada pertahanan zero-day. Kontraktor pertahanan harus merencanakan persistensi tingkat negara.
4. **Latih responsnya.** Mengetahui log mana yang harus diperiksa saat insiden lebih penting daripada memiliki semua alat yang mungkin ada.

## Lihat juga

- [Triad CIA](cia-triad.md)
- [Jenis Malware](malware.md)
- [Rekayasa Sosial](social-engineering.md)
- [Dasar Keamanan Jaringan](network-basics.md)
