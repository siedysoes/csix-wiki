# Dasar Digital Forensics

Digital forensics adalah ilmu untuk mengidentifikasi, mempertahankan, mengumpulkan, dan menganalisis bukti digital agar dapat bertahan saat diuji. Ini penting karena sebuah keputusan incident response ("siapa yang mengambil data tersebut?") hanya sebaik bukti yang mendukungnya, dan penanganan bukti yang salah dapat merusak penyelidikan sekaligus seluruh proses hukum.

## Forensics vs. incident response

Kedua disiplin ini saling tumpang tindih, tetapi memiliki tujuan dan tempo yang berbeda.

| Aspek | Incident Response | Digital Forensics |
| --- | --- | --- |
| Tujuan utama | Mengandung dan memulihkan dengan cepat | Mempertahankan dan membuktikan apa yang terjadi |
| Tekanan waktu | Tinggi (menit hingga jam) | Lebih rendah (akurasi di atas kecepatan) |
| Pertanyaan khas | "Apakah penyerang masih berada di dalam?" | "Bagaimana file itu keluar dari sistem?" |
| Penanganan bukti | Tangkap dulu, dokumentasikan kemudian | Dokumentasikan sebelum menyentuh apa pun |
| Keluaran | Tindakan kontainmen, laporan IR | Timeline, laporan artefak, bukti siap pengadilan |

Dalam praktiknya Anda melakukan keduanya: ambil bukti yang paling volatil terlebih dahulu (order of volatility), lalu lanjutkan ke tahap forensik yang lebih lambat dan sistematis.

## Chain of custody

Chain of custody (rantai penjagaan bukti) adalah catatan tanpa celah tentang siapa yang menangani bukti, kapan, dan mengapa. Jika ada satu mata rantai yang hilang, bukti dapat disangkal dan dikesampingkan.

Untuk setiap barang bukti, catat:

- ID bukti unik dan deskripsinya (asset tag, nomor seri).
- Tanggal, waktu, dan lokasi pengumpulan.
- Siapa yang mengumpulkannya, siapa yang menanganinya berikutnya (setiap serah terima ditandatangani).
- Kondisi penyimpanan dan semua akses sejak pengumpulan.
- Nilai hash (SHA-256) saat pengumpulan dan sebelum setiap sesi analisis.

Simpan bukti asli dalam mode read-only; selalu bekerja pada salinan yang telah diverifikasi.

## Order of volatility (ringkasan)

Kumpulkan data yang paling volatil terlebih dahulu, karena data tersebut hilang saat daya mati:

1. Register dan cache CPU.
2. RAM (proses yang berjalan, koneksi jaringan, kunci yang sudah didekripsi).
3. Status jaringan (tabel ARP, tabel routing, koneksi aktif).
4. Artefak disk (file sementara, sesi browser).
5. File log dan konfigurasi.
6. Backup terarsip dan penyimpanan dingin.

Alat penangkapan memori antara lain DumpIt dan LiME (Linux), yang biasanya dijalankan sebelum sistem dimatikan.

## Forensic imaging

Forensic imaging (imaging forensik) membuat salinan bit-ke-bit dari media penyimpanan sehingga berkas asli bisa disegel. Alat klasiknya adalah `dd`; alat berbasis GUI seperti FTK Imager dan `ewfcreate` (format E01 EnCase) menambahkan kompresi dan metadata tertanam.

```bash
# Linux: image a device to a file, hashing as we go
sudo dd if=/dev/sdb of=/home/evidence/usb-image.dd bs=4M status=progress conv=noerror,sync

# Verify integrity with SHA-256
sha256sum /dev/sdb        # hash the source (before imaging, or a second pass)
sha256sum /home/evidence/usb-image.dd
```

Aturan praktis:

- Gunakan `conv=noerror,sync` agar sektor yang tidak terbaca tidak menggagalkan proses salin.
- Tulis image ke media fisik yang berbeda dari sumbernya.
- Catat kedua hash di log chain of custody; keduanya harus cocok (untuk image `dd`, hash sumber dihitung terpisah karena padding mengubah panjang pada operasi `noerror`).
- Utamakan format E01/AFF untuk kasus nyata: keduanya menanam nilai hash dan mengompresi.

## Timeline analysis

Timeline analysis (analisis linimasa) merekonstruksi peristiwa lintas artefak dengan menormalkan timestamp menjadi satu tampilan kronologis. Perhatikan perbedaan UTC vs. waktu lokal serta entri yang sengaja diubah (sebuah file yang "dibuat" sebelum disk diformat adalah tanda bahaya).

Sumber timestamp: waktu MAC file (modify, access, change), waktu LastWrite key registry, entri log, riwayat browser, header email, catatan koneksi perangkat USB.

Alat: plaso/log2timeline (membuat super-timeline), Timesketch untuk tinjauan kolaboratif, atau pengurutan sederhana di spreadsheet untuk kasus kecil.

```bash
# Generate a timeline from a disk image
log2timeline.py --storage-file case.plaso --image /evidence/usb-image.dd
psort.py -o csv -w case-timeline.csv case.plaso "date >= '2026-01-01'"
```

## Sumber artefak

| Pertanyaan | Artefak | Tempat memeriksa |
| --- | --- | --- |
| Situs apa saja yang dikunjungi? | Riwayat browser | Chrome: `History` (SQLite); Firefox: `places.sqlite` |
| Apa yang diketik di kolom pencarian? | Catatan pencarian/autocomplete | Tabel kata kunci riwayat browser, Windows Search |
| Apa yang dijalankan dan kapan? | Artefak eksekusi | Windows Registry `UserAssist`, `Amcache`, `BAM` |
| Apakah ada flashdisk USB yang dipasang? | Jejak perangkat USB | Windows Registry `SYSTEM\CurrentControlSet\Enum\USBSTOR` |
| Apakah sebuah akun melakukan autentikasi? | Log autentikasi | Windows Event ID 4624/4625; Linux `/var/log/auth.log` |
| Apa yang berubah di sistem? | Jurnal sistem berkas | NTFS `$MFT`, jurnal ext4, `$UsnJrnl:$J` |
| Apakah data diekfiltrasi? | Log jaringan | Log proxy, log firewall, riwayat kueri DNS |

## Forensik memori vs. disk

| Dimensi | Forensik memori (RAM) | Forensik disk |
| --- | --- | --- |
| Volatilitas | Hilang saat daya mati | Bertahan |
| Kecepatan | Snapshot yang bergerak cepat | Catatan historis |
| Mengungkapkan | Koneksi aktif, kunci, kode yang disuntikkan, kata sandi dalam bentuk teks biasa | File, registry, log, riwayat eksekusi |
| Alat khas | Volatility 3, plugin bergaya Rekall | Autopsy, The Sleuth Kit, FTK Imager |
| Risiko utama | Penangkapan tidak lengkap | Manipulasi timestamp |

Penyelidikan yang seimbang menggunakan keduanya: memori menunjukkan apa yang sedang dilakukan proses, disk menunjukkan apa yang ditinggalkannya.

## Kesadaran anti-forensics

Penyerang mungkin berusaha menutupi jejaknya; mengetahui tekniknya membantu Anda mendeteksinya:

- **Timestomping**: mengubah waktu MAC (misalnya, Windows `timestomp`, Linux `touch -d`). Deteksi lewat ketidakcocokan entri MFT antara atribut `$STANDARD_INFORMATION` dan `$FILE_NAME`, serta dengan membandingkan data jurnal atau backup.
- **Secure delete / wiping**: alat seperti `shred` menimpa isi file. Pada SSD dengan wear leveling, blok asli mungkin masih ada di area yang belum dipetakan — tetapi "SSD selalu menyimpan segalanya" juga mitos: SSD yang terenkripsi membuang data saat proses ATA erase atau crypto-erase terjadi. Perlakukan penimpaan sebagai alasan untuk mencari salinan jurnal, snapshot, dan backup, bukan menganggap data sudah tidak dapat dipulihkan.
- **Pembersihan log**: menghapus `/var/log` atau mengosongkan Windows Security log (Event ID 1102 mencatat pembersihan itu sendiri).
- **Teknik fileless**: aktivitas yang hanya hidup di memori, sehingga nilai penangkapan RAM semakin tinggi.
- **Slack space dan alternate data streams**: menyembunyikan data di tempat yang tidak dicari oleh salinan biasa.

## Skenario terapan: memulihkan file yang terhapus

Tujuan: memulihkan file kontrak PDF yang dihapus pengguna dari sebuah workstation Windows.

1. **Otorisasi dan dokumentasikan.** Konfirmasi cakupan secara tertulis, buka catatan chain of custody, catat nomor seri workstation dan waktu saat ini.
2. **Tangkap data volatil.** Jalankan penangkapan RAM (misalnya DumpIt) selama mesin menyala; catat handle yang terbuka dan koneksi jaringan.
3. **Buat image disk.** Dengan mesin mati, sambungkan drive dalam mode read-only (write blocker perangkat keras lebih disukai) dan buat image dengan FTK Imager atau `dd`, sambil menghitung SHA-256 dari image tersebut.
4. **Bekerja pada salinan.** Mount atau muat image di Autopsy/FTK; jangan pernah menganalisis berkas asli.
5. **Lokalkan file.** Periksa Recycle Bin (`$Recycle.Bin`, metadata berada di file `$I`, data di file `$R`). Jika tidak ada di sana, parse `$MFT` untuk mencari entri yang tidak dialokasikan: nama file tetap ada meskipun sudah dihapus.
6. **Pulihkan data.** Jika MFT masih menunjuk ke klaster yang valid, ekspor file tersebut dan verifikasi apakah bisa dibuka. Jika klaster sudah sebagian ditimpa, coba gunakan carver (Autopsy, TestDisk, PhotoRec) pada ruang yang tidak dialokasikan.
7. **Verifikasi dan laporkan.** Hash file yang dipulihkan, catat waktu MAC-nya dan kondisi klaster, serta dokumentasikan setiap langkah agar pemeriksa lain dapat mengulangi hasil yang sama.

Pemulihan sering berhasil karena penghapusan biasanya hanya menandai ruang sebagai bebas; isinya bertahan sampai sistem operasi menggunakan kembali klaster tersebut.

## Lihat juga

- [Dasar Incident Response](incident-response.md)
- [Membangun Home Lab](hands-on-lab.md)
- [Keamanan Sistem Operasi](../fundamentals/operating-system-security.md)
- [Lanskap Ancaman](../fundamentals/threat-landscape.md)
