# Dasar Incident Response

Incident response adalah praktik mendeteksi, menahan, memberantas, dan pulih dari insiden keamanan secara terkendali dan terdokumentasi. Proses yang terdefinisi itu penting karena jam-jam pertama selalu kacau — tanpanya, tim menghancurkan bukti, melewati batas waktu notifikasi, dan mengambil keputusan yang tak bisa dijelaskan siapa pun setelahnya.

## Apa yang dihitung sebagai insiden

**Event** adalah peristiwa apa pun yang bisa diamati di sistem atau jaringan. **Insiden** adalah event yang merugikan, atau mengancam merugikan, kerahasiaan, integritas, atau ketersediaan informasi — dan karena itu membutuhkan penanganan terkoordinasi di luar operasi rutin.

Insiden mencakup:

- Malware atau ransomware di workstation atau server
- Akses tanpa izin yang terkonfirmasi, login phishing yang berhasil, atau kredensial yang bocor
- Exfiltrasi data atau akses tanpa izin ke data sensitif
- Defacement situs web, denial of service, atau gangguan yang disebabkan serangan
- Bucket penyimpanan publik yang membocorkan data pelanggan

Tidak setiap event adalah insiden: upaya phishing yang diblokir, satu kali login gagal, atau kerentanan yang di-patch tepat waktu layak dicatat tetapi tidak layak membangunkan siapa pun. Ujiannya adalah apakah penanganannya membutuhkan orang, keputusan, dan bukti — bukan apakah peringatannya berisik.

## Siklus hidup NIST SP 800-61

NIST SP 800-61 (Revisi 2) menggambarkan siklus hidup empat fase; Revisi 3, terbit pada 2025, menyusun ulang pedoman tersebut mengikuti NIST CSF 2.0 sambil mempertahankan aktivitas praktis yang sama. Anggap fase-fase itu sebagai lingkaran: temuan dari aktivitas pasca-insiden langsung kembali ke tahap persiapan.

| Fase | Tujuan | Aktivitas umum |
|---|---|---|
| 1. Preparation | Siap sebelum apa pun terjadi | Playbook, pelatihan, tabletop exercise, cakupan log dan EDR/SIEM, kontak dan jalur eskalasi, kapasitas cadangan |
| 2. Detection and analysis | Menemukan insiden dan memahami cakupannya | Triase peringatan, tinjauan log dan paket, rekonstruksi linimasa, penetapan cakupan host dan data yang terdampak, penilaian tingkat keparahan |
| 3. Containment, eradication and recovery | Menghentikan penyebaran, menghilangkan penyebab, memulihkan layanan | Isolasi, pemblokiran indikator, penghapusan malware dan persistence, patch pada vektor akses awal, pemulihan cadangan yang diketahui baik, pemantauan terhadap kejadian berulang |
| 4. Post-incident activity | Belajar dan memperbaiki | After-action review, lessons learned, pembaruan deteksi dan playbook, metrik, penyimpanan bukti, notifikasi hukum dan regulasi |

Fase-fase itu tidak sepenuhnya berurutan. Containment sering dimulai selagi analisis masih berjalan, dan pemulihan dapat mengungkap bukti baru yang mengembalikan tim ke tahap deteksi.

## Jam pertama

Daftar periksa praktis untuk 60 menit pertama setelah laporan masuk:

1. **Konfirmasi dan catat waktu.** Rekam kapan laporan tiba dan siapa yang melaporkan, dalam UTC.
2. **Triase tingkat keparahan.** Apakah produksi down? Apakah data terpapar? Apakah penyerang masih aktif?
3. **Nyatakan insiden.** Tetapkan seorang incident commander dan buka satu kanal insiden (bridge, ruang obrolan, tiket).
4. **Lindungi bukti lebih dulu.** Tangkap data volatil — memori, lalu keadaan jaringan — sebelum menyentuh disk.
5. **Tahan tanpa menghancurkan bukti.** Isolasikan host yang terdampak di tingkat jaringan (VLAN khusus, port switch, aturan firewall), bukan mematikannya, kecuali host tersebut sedang mengenkripsi data.
6. **Tetapkan cakupan.** Identifikasi sistem, akun, dan data yang terdampak; telusuri log untuk indikator yang sama di tempat lain.
7. **Beritahu.** Eskalasi ke manajemen, hukum, dan petugas privasi ketika data pribadi mungkin terlibat, dan mulai batas waktu regulasi apa pun.
8. **Dokumentasikan terus-menerus.** Scribe memelihara linimasa: tindakan, pelaku, waktu, hasil.

Jangan berkoordinasi lewat sistem yang mungkin sudah disusupi. Jika email atau obrolan perusahaan berada di host yang terdampak, segera beralih ke kanal luar jalur (out-of-band).

## Peran dalam insiden

| Peran | Tanggung Jawab | Catatan |
|---|---|---|
| Incident commander | Memegang keputusan: menyatakan, menahan, mengeskalasi, menutup | Keterampilan teknis bersifat opsional; ketegasan dan komunikasi yang jelas wajib ada |
| Scribe | Memelihara linimasa resmi | Satu orang, menulis terus-menerus — tidak ada orang lain yang mencatat |
| Komunikasi | Menginformasikan eksekutif, staf, pelanggan, dan regulator | Berbicara dari pesan yang telah disetujui, bukan opini pribadi |
| Lead teknis / analis | Menyelidiki, menahan, memberantas, memulihkan | Melapor kepada commander, bukan langsung ke eksekutif |
| Hukum / privasi / SDM | Menasihati soal kewajiban dan urusan personel | Dilibatkan sejak awal ketika data atau karyawan terlibat |

Dalam tim kecil, satu orang mungkin memegang beberapa peran — tetapi namakan peran itu secara eksplisit secara tertulis, karena "siapa yang memutuskan?" adalah pertanyaan yang paling cepat menghentikan respons.

## Bukti: order of volatility

Kumpulkan dari yang paling volatil ke yang paling tidak volatil, karena setiap langkah dapat menghancurkan data di atasnya.

| Urutan | Jenis | Contoh | Cara menangkap |
|---|---|---|---|
| 1 | Memori | Daftar proses, koneksi terbuka, malware yang mengendap di RAM | Memory dump (`winpmem`, `avml`, `LiME`) sebelum reboot apa pun |
| 2 | Keadaan jaringan | Koneksi aktif, cache ARP dan DNS, lalu lintas langsung | `netstat -ano`, `ss -tunap`, `arp -a`, penangkapan paket |
| 3 | Disk dan log | Berkas, registry key, log persisten, image disk | Image forensik dengan write blocker atau `dd`/FTK Imager, lalu hash |

Aturan yang menyertainya: kerjakan pada salinan, jangan pada asli; hash setiap artefak (SHA-256) saat mengumpulkannya; catat siapa mengumpulkan apa dan kapan; jangan pernah menyelidiki langsung di mesin bukti.

## Aturan komunikasi

- Fakta saja, dengan stempel waktu. Pisahkan apa yang Anda amati dari apa yang Anda simpulkan, dan beri label simpulan sebagai hipotesis.
- Satu sumber kebenaran: kanal insiden dan linimasa milik scribe. Tidak ada keputusan sampingan lewat pesan pribadi.
- Need to know: jangan menyiarkan detail kompromi ke seluruh perusahaan; bagikan berdasarkan kebutuhan untuk mengetahui.
- Pernyataan eksternal hanya datang dari kepala komunikasi, disepakati bersama hukum — pelanggan, regulator, dan pers tidak boleh mendengarnya lebih dulu dari seorang insinyur.
- Tulis untuk pembaca yang datang belakangan: auditor, pengadilan, dan penerus Anda.
- Anggap penyerang bisa membaca obrolan internal dan email, dan rencanakan kanalnya dengan demikian.

## Tabletop exercise

Tabletop adalah diskusi terfasilitasi atas skenario yang realistis — tanpa eksploitasi, tanpa sistem langsung, hanya tim yang membicarakan keputusan di atas kertas.

1. Pilih skenario yang sesuai dengan risiko Anda: ransomware di file server, token admin yang dicuri, basis data pelanggan yang terpapar.
2. Tetapkan latar dalam satu paragraf dan sampaikan inject awal (misalnya, tiket help-desk yang melaporkan berkas terenkripsi).
3. Tanyakan kepada tim apa yang akan mereka lakukan — siapa yang menyatakan insiden, apa yang ditangkap lebih dulu, siapa yang diberi tahu, dan kapan batas waktunya.
4. Tambahkan inject setiap 10-15 menit untuk memaksa adanya kompromi: cadangan berumur 48 jam, CEO ingin pernyataan pers.
5. Batasi waktu setiap titik keputusan dan catat di mana tim tersendat atau berbeda pendapat.
6. Tutup dengan after-action review: apa yang tidak jelas, apa yang berubah di playbook, siapa yang bertanggung jawab atas setiap perbaikan.

Jalankan setidaknya setiap kuartal. Latihan dua puluh menit yang mengungkap bahwa Anda tidak punya daftar sistem kritis lebih berharga daripada latihan delapan jam yang memanjakan tim.

## Skenario: jam pertama insiden phishing

Seorang pengguna keuangan melaporkan bahwa setelah membuka tautan email, mereka mengetikkan sandi ke halaman yang mirip sekali dengan aslinya. Scribe membuka linimasa:

```text
09:07 UTC  Report received from finance user (email forwarded to security)
09:10 UTC  Incident declared: IR-014; commander and scribe named, channel opened
09:12 UTC  Password reset and active sessions revoked for the user
09:15 UTC  Sign-in logs pulled: access from 203.0.113.44 at 09:03 UTC
09:18 UTC  Memory and network state captured from the user's workstation
09:25 UTC  Same source address searched across all accounts: one further match
09:35 UTC  Management and privacy officer notified; second account disabled
```

Reset dilakukan sebelum penangkapan memori karena aset yang disusupi adalah kredensial, bukan workstation, dan penyerang sedang menggunakannya saat itu juga. Order of volatility memberi tahu apa yang harus dikumpulkan dari sistem yang akan Anda sentuh — aturan itu tidak pernah mengalahkan upaya menghentikan intrusi yang sedang berlangsung.

## Lihat juga

- [Triad CIA](../fundamentals/cia-triad.md)
- [Lanskap Ancaman](../fundamentals/threat-landscape.md)
- [Jenis-jenis Malware](../fundamentals/malware.md)
- [Dasar Keamanan Jaringan](../fundamentals/network-basics.md)
- [Membangun Home Lab](hands-on-lab.md)
