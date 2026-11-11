# Keamanan Sistem Operasi

Keamanan sistem operasi adalah kumpulan kontrol yang melindungi sebuah host — penggunanya, file, proses, dan koneksi jaringannya — dari akses tanpa izin dan kompromi. Baik Anda mengamankan laptop, server, maupun instance cloud, prinsip inti yang sama berlaku: batasi siapa boleh melakukan apa, jaga sistem tetap ter-patch, catat apa yang terjadi, dan enkripsi hal yang penting.

## Least Privilege

Least privilege berarti setiap pengguna, proses, dan layanan berjalan dengan izin minimum yang dibutuhkan untuk menjalankan tugasnya, tidak lebih.

- Jangan menggunakan akun administrator untuk pekerjaan sehari-hari.
- Layaran sebaiknya berjalan sebagai akun khusus dengan privilege rendah, bukan `root` atau `SYSTEM`.
- Hapus perangkat lunak dan akun yang tidak lagi dibutuhkan — setiap izin ekstra adalah jalur serangan ekstra.
- Berikan akses tinggi (elevated) tepat waktu (just in time), bukan permanen.

Di Linux ini ditegakkan dengan pengguna, grup, dan `sudo`; di Windows dengan User Account Control (UAC), keanggotaan grup, dan privilege rights. Penyerang yang membajak proses non-admin jauh lebih terbatas dibanding yang mewarisi hak admin.

## Patching

Sistem tanpa patch adalah titik masuk paling andal bagi penyerang, dan beberapa cacat bahkan bisa dijangkiti worm. Contoh historis termasuk MS17-010 (EternalBlue, dipakai oleh WannaCry) dan cacat remote code execution Apache Log4j **CVE-2021-44228** (Log4Shell).

Panduan patching praktis:

1. Jaga inventaris aset yang akurat — Anda tidak bisa mem-patch apa yang tidak Anda ketahui.
2. Berlangganan security advisory vendor untuk OS dan perangkat lunak utama Anda.
3. Tetapkan SLA: cacat kritis/aktif dieksploitasi dalam hitungan hari, pembaruan rutin dalam siklus bulanan.
4. Uji sebelum penerapan luas, tetapi jangan biarkan pengujian menunda patch darurat.
5. Reboot bila diperlukan; patch kernel yang belum diterapkan sama saja dengan tidak diterapkan.

## Firewall Host

Firewall host melindungi satu mesin bahkan ketika perimeter jaringan gagal.

```powershell
# Windows: blokir inbound SMB kecuali dari subnet manajemen
New-NetFirewallRule -DisplayName "Block SMB Inbound" -Direction Inbound `
  -Protocol TCP -LocalPort 445 -Action Block
```

```bash
# Linux (ufw): izinkan hanya SSH dan web, tolak sisanya
sudo ufw default deny incoming
sudo ufw allow 22/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

Default deny untuk inbound, izin eksplisit untuk layanan yang diperlukan. Ide yang sama ditemukan di cloud security group.

## Enkripsi Disk

Enkripsi seluruh disk (full-disk encryption) melindungi data saat perangkat hilang atau dicuri, karena disk tidak bisa dibaca di luar mesin tanpa kunci.

| OS | Alat | Metode kunci/pembukaan umum |
|---|---|---|
| Windows Pro/Enterprise | BitLocker | TPM + PIN atau recovery password |
| macOS | FileVault 2 | Password pengguna / recovery key |
| Linux | LUKS (cryptsetup) | Passphrase atau keyfile |
| Mobile | Android FBE / iOS Data Protection | Kunci turunan dari device passcode |

Batasan penting:

- Enkripsi melindungi data **at rest**. Enkripsi tidak berdaya terhadap malware yang berjalan di sesi yang sudah terbuka dan login.
- Simpan recovery key di tempat yang aman (Entra ID/AD, password manager). Kunci yang tidak bisa dipulihkan adalah outage yang menimpa diri sendiri.
- Di Linux, LUKS melindungi volume; `/home` di partisi yang tidak terenkripsi atau swap masih bisa bocor kecuali dienkripsi terpisah.

## Windows vs Linux: Akun dan Izin

| Aspek | Windows | Linux |
|---|---|---|
| Penyimpanan identitas | SAM lokal, domain Active Directory | `/etc/passwd`, LDAP/SSSD |
| Superuser | `Administrator` / `SYSTEM` | `root` (atau pengguna dengan sudo) |
| Model izin | ACL pada file/objek; NTFS | Bit owner/group/other ditambah ACL (`setfacl`) |
| Elevasi | Prompt UAC, `runas` | `sudo` dengan `/etc/sudoers` yang dikonfigurasi |
| Akun layanan | Akun virtual (`NT SERVICE\...`) | Unit systemd dengan `User=` |
| Audit | Log keamanan dengan SACL | `auditd` |

Pemeriksaan sehari-hari yang berguna:

```bash
# Siapa yang punya hak sudo?
sudo grep -E '^(root|%sudo)' /etc/sudoers
# Siapa yang sedang login dan sejak kapan?
who
```

```powershell
# Daftar administrator lokal
Get-LocalGroupMember -Group "Administrators"
```

## Logging dan Audit

Anda tidak bisa menyelidiki apa yang tidak pernah Anda catat.

- **Windows Event Viewer**: log Security (logon/logoff 4624/4625, penggunaan privilege, perubahan kebijakan), log System, dan log PowerShell. Pusatkan dengan Windows Event Forwarding atau SIEM.
- **Linux journald**: `journalctl -u ssh -f` untuk log layanan langsung; penyimpanan persisten memerlukan `/var/log/journal`. Untuk audit syscall yang lebih dalam gunakan `auditd`.
- **Retensi**: simpan log cukup lama untuk mendeteksi insiden yang bergerak lambat, sinkronkan waktu dengan NTP, dan batasi akses tulis agar penyerang tidak bisa menghapus jejak.

```
$ journalctl -u ssh --since "2 hours ago" | grep -i failed
Nov 11 02:14:03 srv01 sshd[812]: Failed password for invalid user admin ...
```

Kegagalan berulang seperti ini adalah sinyal brute force Anda — jika ada yang memantau.

## Mitos Penghapusan Aman

Miskonsepsi umum yang layak ditinggalkan:

- **Menghapus file atau mengosongkan recycle bin** hanya memutus tautan pointer; data tetap ada sampai ditimpa.
- **Satu kali overwrite sudah cukup pada SSD modern.** Untuk HDD, satu kali overwrite penuh umumnya sudah cukup; standar DoD "tujuh kali overwrite" yang lama tidak diperlukan.
- **`shred` menjamin penghapusan.** Pada SSD dengan wear leveling, sistem berkas berjournal, atau RAID, overwrite perangkat lunak tidak bisa menjamin setiap salinan hancur. Ruang kosong juga tidak terlihat oleh sistem berkas.
- **Enkripsi adalah jawaban yang sesungguhnya.** Dengan enkripsi seluruh disk, "penghapusan aman" menjadi menghancurkan kunci — itulah mengapa pembuangan berbasis enkripsi menjadi pendekatan yang direkomendasikan untuk SSD.

Untuk media yang sudah dipensiunkan, gunakan perintah secure-erase dari vendor (ATA/NVMe sanitize), hancurkan kuncinya, atau fisik-hancurkan drive.

## Baseline Hardening dengan CIS Benchmark

**CIS Benchmark** adalah standar konfigurasi konsensus dari Center for Internet Security yang menggambarkan pengaturan yang sudah diperkuat untuk OS, aplikasi, atau layanan cloud tertentu, disusun dalam implementation group.

Alur kerja yang umum:

1. Pilih aset Anda (misalnya, *CIS Microsoft Windows 11 Benchmark*).
2. Tentukan profil: Level 1 (dapat dipertahankan, dampak performa minimal) vs Level 2 (keamanan lebih tinggi, biaya operasional lebih besar).
3. Pindai dengan sebuah alat (Microsoft Baseline Configuration Analyzer, OpenSCAP, Nessus, atau MDM Anda).
4. Perbaiki (remediate), dokumentasikan pengecualian dengan alasan bisnis, dan pindai ulang secara berkala.

Contoh: sebuah aturan Linux Level 1 mungkin mensyaratkan `net.ipv4.conf.all.send_redirects = 0`. Gagal pada aturan itu adalah temuan; menonaktifkan layanan yang Anda butuhkan adalah pengecualian yang menjadi tanggung jawab Anda.

## Lihat juga

- [Triad CIA](cia-triad.md)
- [Dasar Keamanan Jaringan](network-basics.md)
- [Password & Autentikasi](passwords-authentication.md)
- [Jenis Malware](malware.md)
- [Peta Belajar](../career/study-roadmap.md)
