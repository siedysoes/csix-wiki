# Dasar Keamanan Cloud

Keamanan cloud adalah praktik melindungi data, aplikasi, dan infrastruktur yang di-host di cloud publik dan privat. Karena layanan cloud berjalan di atas infrastruktur bersama dan dapat dijangkau lewat internet secara default, sebagian besar pelanggaran cloud berasal dari konfigurasi yang salah dan akses yang terlalu luas, bukan dari serangan yang belum pernah ada sebelumnya.

## Model tanggung jawab bersama

Penyedia mengamankan cloud; Anda mengamankan apa yang Anda masukkan ke dalamnya. Di mana batasnya jatuh tergantung pada model layanannya — inilah sumber kepercayaan keliru (false confidence) yang paling umum dalam keamanan cloud.

| Area | IaaS (VM, storage, jaringan) | PaaS (platform terkelola, serverless) | SaaS (email, CRM, kolaborasi) |
|---|---|---|---|
| Pusat data fisik dan perangkat keras | Penyedia | Penyedia | Penyedia |
| Infrastruktur jaringan dan hypervisor | Penyedia | Penyedia | Penyedia |
| Patch OS pada host dasar | Penyedia | Penyedia | Penyedia |
| OS guest, runtime, middleware | Pelanggan | Penyedia | Penyedia |
| Aplikasi dan kode | Pelanggan | Pelanggan | Penyedia (Anda mengonfigurasikannya) |
| Identitas, akses, dan kunci | Pelanggan | Pelanggan | Pelanggan |
| Klasifikasi data dan enkripsi | Pelanggan | Pelanggan | Pelanggan |
| Konfigurasi jaringan (firewall, security group) | Pelanggan | Pelanggan (sebagian) | Pelanggan (pengaturan akses) |

Di setiap model, Anda memegang tiga pekerjaan yang sama: siapa yang bisa masuk (IAM), apa yang mereka boleh lakukan (izin), dan ke mana data pergi (konfigurasi dan enkripsi).

## Konfigurasi keliru yang paling umum

| Konfigurasi keliru | Apa yang salah | Perbaikan pertama |
|---|---|---|
| Object storage publik | Bucket yang berisi ekspor, cadangan, atau kredensial menjadi dapat dibaca siapa pun yang punya URL | Blokir akses publik di tingkat akun, lalu tinjau kebijakan per bucket |
| IAM terlalu luas | Pengguna atau layanan dengan `*` di semua hal; satu kompromi menjadi pengambilalihan akun penuh | Kebijakan least-privilege, hapus peran yang tidak terpakai, tinjau izin secara berkala |
| Tanpa MFA | Akses konsol dan API hanya berbasis sandi; satu kredensial hasil phishing sudah cukup | Wajibkan MFA untuk semua pengguna manusia, administrator lebih dulu |
| Security group / aturan firewall terbuka | `0.0.0.0/0` pada SSH (22), RDP (3389), atau port basis data | Batasi ke rentang yang diketahui atau jump host; jangan pernah mengekspos basis data |
| Access key jangka panjang | Kunci statis yang di-commit ke kode atau disimpan di laptop, berlaku bertahun-tahun, jarang dirotasi | Peran dan identitas beban kerja jangka pendek; rotasi dan audit apa yang tersisa |

Sebagian besar hal ini terlihat dalam tinjauan lima menit terhadap halaman storage, IAM, dan jaringan akun. Dan justru itulah sebabnya sering terlewat — tidak ada yang menugaskan tinjauan lima menit itu.

## IAM: least privilege dan peran

Mulai dari "tanpa akses" dan hanya tambahkan apa yang dibutuhkan sebuah tugas.

- **Peran untuk beban kerja, bukan kunci.** Berikan IAM role kepada VM, kontainer, dan pipeline CI agar cloud menerbitkan kredensial jangka pendek secara otomatis, alih-alih Anda menanamkan kunci statis.
- **Identitas terfederasi untuk manusia.** Single sign-on dari penyedia identitas Anda dengan MFA; tanpa akun cloud lokal untuk orang.
- **Batasi setiap kebijakan** berdasarkan sumber daya, aksi, dan kondisi — IP sumber, tag wajib, MFA aktif.
- **Pisahkan lingkungan.** Akun atau langganan berbeda untuk produksi, staging, dan sandbox; jangan pernah berbagi kredensial admin di antaranya.
- **Akun admin break-glass.** Satu akun darurat yang dipantau ketat, nonaktif secara default, hanya dipakai ketika akses normal gagal.
- **Tinjau secara berkala.** Tinjauan izin setiap kuartal; hapus peran dan kunci yang tidak bisa dipertanggungjawabkan siapa pun.

Tinjauan akses bisa dimulai dari CLI penyedia itu sendiri:

```bash
# What policies are attached to a workload role?
aws iam list-attached-role-policies --role-name app-role

# Is this bucket reachable by the public?
aws s3api get-bucket-policy-status --bucket company-exports

# Does the root account have MFA enabled? (1 = yes, 0 = no)
aws iam get-account-summary --query 'SummaryMap.AccountMFAEnabled'
```

Kesalahan dari `get-bucket-policy-status` berarti bucket tidak memiliki kebijakan — yang tidak sama dengan privat. Akses publik juga bisa berasal dari ACL atau access point bersama, jadi periksa juga pengaturan "block public access" di tingkat akun.

## Logging dan jejak audit cloud

Setiap penyedia besar mencatat aktivitas control plane: siapa yang membuat, mengubah, atau menghapus apa, kapan, dan dari alamat mana. Log aktivitas (jejak audit) ini adalah bukti utama Anda baik untuk configuration drift maupun investigasi insiden.

- Aktifkan layanan log audit penyedia (CloudTrail, Activity Log, Activity Tracker) di **setiap** region atau langganan — pencatatan biasanya per-region dan nonaktif secara default.
- Alirkan log ke akun atau lokasi penyimpanan terpisah yang tidak bisa dihapus oleh administrator beban kerja; penyerang umumnya menutup jejak dengan mematikan pencatatan lebih dulu.
- Buat peringatan pada event bermakna tinggi: pencatatan dimatikan, pembuatan IAM user atau role baru, kebijakan bucket yang dibuat publik, security group yang dibuka ke seluruh dunia, MFA yang dilepas.
- Kumpulkan juga log data plane (log akses objek, log audit basis data): log control plane menunjukkan sebuah sumber daya diubah, bukan apa yang dibaca keluar darinya.

## Infrastructure as code dan configuration drift

Mendefinisikan infrastruktur dalam kode — Terraform, CloudFormation, Pulumi, Bicep — membuat tinjauan keamanan mungkin dilakukan sebelum apa pun ada: kebijakannya hidup di pull request, bukan di klik konsol.

Configuration drift adalah apa yang terjadi ketika kenyataan menjauh dari kode: seseorang memperbaiki produksi lewat konsol, dan `apply` berikutnya diam-diam mengembalikannya atau gagal. Dua kontrol menjaga hal ini tetap jujur:

1. **Semuanya lewat pipeline.** Perubahan lewat konsol adalah pengecualian, dicatat dan ditulis kembali ke kode pada hari yang sama.
2. **Drift detection.** Jalankan drift detection milik penyedia atau `terraform plan` secara berkala dan perlakukan setiap perbedaan tak terduga sebagai temuan.

Memindai kode sebelum deploy menangkap masalah yang akan ada di mana-mana; memindai akun yang berjalan menangkap masalah yang sudah ada. Anda membutuhkan keduanya, karena salah satunya saja meninggalkan titik buta.

## Skenario: bucket ekspor yang terpapar

Seorang magang membuat bucket bernama `company-exports` untuk berbagi CSV dengan kontraktor, mengaturnya ke public-read, dan melanjutkan pekerjaannya. Tidak ada yang rusak, jadi tidak ada yang menyadari — sampai sebuah scanner atau log akses objek menandainya.

1. **Deteksi.** CSPM scan atau log aktivitas storage menunjukkan pemberian izin publik, atau `get-bucket-policy-status` mengembalikan `IsPublic: true`.
2. **Tahan.** Nyalakan block public access di tingkat akun (satu pengaturan, semua bucket), lalu hapus ACL atau kebijakan yang spesifik.
3. **Nilai.** Baca log akses objek untuk melihat apakah ada yang diunduh. Jawaban itu menentukan apakah ini insiden dengan kewajiban notifikasi atau hanya near miss.
4. **Perbaiki penyebabnya.** Tawarkan jalur berbagi yang disetujui — presigned URL atau layanan transfer file terkelola — agar solusi sementara itu tidak kembali bulan depan.
5. **Cegah.** Berikan bucket sebuah tag pemilik dan tambahkan pemeriksaan kebijakan ke pipeline IaC agar pembuatan yang sama gagal tinjauan lain kali.

## CSPM dalam satu paragraf

Alat Cloud Security Posture Management (CSPM) terus-menerus membaca konfigurasi cloud Anda dan membandingkannya dengan baseline praktik terbaik — panduan hardening penyedia, CIS Benchmark, dan kebijakan Anda sendiri — lalu melaporkan (dan terkadang memperbaiki otomatis) penyimpangan seperti bucket publik, security group terbuka, atau disk yang tidak terenkripsi. CSPM tidak menghentikan penyerang yang sudah berada di dalam beban kerja dan tidak memperbaiki kode yang rentan; CSPM mengecilkan jarak antara konfigurasi yang diinginkan dan yang sebenarnya, dan di situlah sebagian besar pelanggaran cloud bermula.

## Daftar periksa hardening

- [ ] MFA diwajibkan untuk semua pengguna manusia; tanpa access key jangka panjang untuk orang
- [ ] Akun root / break-glass dikunci ketat, kredensial disimpan luar jaringan, penggunaan diberi peringatan
- [ ] Peran least-privilege untuk beban kerja; tanpa kunci statis di kode, image, atau notebook
- [ ] Object storage: akses publik tingkat akun diblokir, enkripsi aktif secara default
- [ ] Security group dan firewall: tanpa `0.0.0.0/0` pada port manajemen atau basis data
- [ ] Log audit diaktifkan di mana pun dan dikirim ke akun terpisah, dengan peringatan pada event log-disable
- [ ] Enkripsi at rest aktif secara default untuk disk, bucket, dan basis data; TLS diwajibkan saat transit
- [ ] Penyedia identitas terpusat dengan akses bersyarat dan proses offboarding
- [ ] Inventaris sumber daya dengan penandaan wajib — Anda tidak bisa mengamankan aset yang tidak diketahui
- [ ] Pipeline IaC dengan pemeriksaan kebijakan dan drift detection terjadwal
- [ ] Vulnerability scanning untuk image dan beban kerja, dengan pemilik yang ditetapkan per temuan
- [ ] Cadangan di luar akun produksi, diuji dengan pemulihan sungguhan

## Lihat juga

- [Triad CIA](../fundamentals/cia-triad.md)
- [Dasar Keamanan Jaringan](../fundamentals/network-basics.md)
- [Kata Sandi & Autentikasi](../fundamentals/passwords-authentication.md)
- [Manajemen Kerentanan](vulnerability-management.md)
