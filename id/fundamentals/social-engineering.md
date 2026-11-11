# Rekayasa Sosial

Rekayasa sosial (social engineering) adalah manipulasi terhadap orang agar melakukan tindakan atau mengungkapkan informasi yang melemahkan keamanan, alih-alih menyerang teknologi secara langsung. Teknik ini berhasil karena mengeksploitasi sifat manusia yang wajar — kepercayaan, rasa mendesak, keramahan — sehingga kontrol teknis saja tidak bisa menghentikannya.

## Teknik-teknik utama

| Teknik | Kanal | Trik inti | Tujuan khas |
|---|---|---|---|
| Phishing | Email massal | Pesan palsu tetapi meyakinkan berisi tautan atau lampiran | Mencuri kredensial atau memasang malware |
| Spear phishing | Email terarah | Dipersonalisasi untuk satu orang menggunakan detail hasil riset | Mengkompromikan satu akun bernilai tinggi |
| Vishing | Panggilan telepon | Penelepon mengaku sebagai TI, bank, atau pejabat eksekutif | Memancing kode, kata sandi, atau pembayaran |
| Smishing | SMS / aplikasi pesan | Pesan pendek berisi tautan mendesak atau nomor untuk dihubungi balik | Pencurian kredensial atau pemasangan malware |
| Business email compromise (BEC) | Email | Menyamar sebagai eksekutif, vendor, atau mitra | Mengalihkan pembayaran ke rekening penyerang |
| Pretexting | apa saja, biasanya suara atau email | Skenario karangan di mana penyerang punya "alasan" | Mendapatkan catatan, akses, atau data sensitif |
| Baiting | Fisik atau digital | Umpan yang menggoda (USB gratis, berkas "bocoran") | Memancing korban menjalankan malware atau mengunjungi jebakan |
| Quid pro quo | Telepon / tatap muka | "Saya bantu kalau Anda beri saya..." | Mendapatkan kredensial atau akses fisik |
| Tailgating | Fisik | Mengikuti orang yang berwenang melewati pintu | Masuk area terbatas tanpa lencana |

## Keluarga phishing secara rinci

**Phishing** bersifat luas: ribuan email identik yang mengklaim paket belum terkirim atau kata sandi akan kedaluwarsa. Ia bekerja lewat volume — satu persen yang mengklik sudah cukup.

**Spear phishing** menyasar satu target. Penyerang membaca siaran pers perusahaan dan profil LinkedIn, lalu mengemail manajer keuangan dengan menyebut sebuah kesepakatan nyata. Tingkat keberhasilannya jauh lebih tinggi karena tidak ada yang terlihat generik.

**Vishing** bergeser ke suara, di mana verifikasi lewat panggilan balik jauh lebih sulit. Varian klasiknya adalah panggilan palsu "dukungan TI" pada hari Senin yang sibuk: "Kami melihat kesalahan di akun Anda, bisa bacakan kode yang baru saja kami kirimkan lewat SMS?" Penyerang — yang telah memicu prompt MFA yang asli — menerima kodenya.

**Smishing** adalah gagasan yang sama lewat SMS, sering menyamar sebagai layanan pengiriman atau bank dengan tautan pendek.

**BEC** biasanya tidak melibatkan malware sama sekali. Penyerang mempelajari format faktur, lalu mengirim email yang tampak bersih:

```text
From: cfo@your-company.example        (displayed name: CFO's real name)
To: accounts.payable@your-company.example
Subject: Re: Invoice #4471 - urgent, supplier bank changed

Hi team,
Please update our supplier's bank details before today's run —
the account change was confirmed on the call this morning.
New IBAN: XX00 1234 5678 9012 3456
Send the confirmation reply to this thread only.
Thanks,
CFO
```

Trik reply-to, desakan waktu, dan instruksi "jangan dibicarakan di tempat lain" semuanya adalah penanda BEC yang standar. Pada kebocoran Twitter 2020, rekayasa sosial terhadap staf — bukan cacat perangkat lunak — membantu penyerang mengambil alih akun-akun profil tinggi.

## Pretexting, baiting, dan quid pro quo

- **Pretexting** adalah konteks karangan yang membuat permintaan terasa seperti urusan rutin. Penyerang menelepon mengaku dari HR yang sedang mengumpulkan formulir pajak terbaru, atau mengaku pemilik rumah yang meminta kunci apartemen. Keterampilan kuncinya adalah riset: semakin banyak detail yang dimuat pretext, semakin sedikit ia dipertanyakan.
- **Baiting** menawarkan sesuatu yang menggoda. Secara fisik: USB terinfeksi berlabel "Payroll Q4" yang ditinggalkan di parkiran; secara digital: berkas bernama `salary_adjustment.xlsx` di drive bersama. Rasa penasaran mengerjakan sisanya.
- **Quid pro quo** mempertukarkan sebuah bantuan: "Kami tim keamanan yang sedang melakukan audit kata sandi — bacakan kata sandi Anda yang sekarang dan kami akan meng-upgrade Anda ke skema baru." Berbeda dengan darurat palsu pada vishing, teknik ini berpura-pura sebagai sebuah layanan.

## Teknik fisik

**Tailgating** (atau piggybacking) adalah mengikuti seorang karyawan melewati pintu yang terkunci lencana, sering sambil tangan penuh kotak dan sapaan ramah "bisa tolong bukakan?". **Mengunci celah tailgating** berarti menerapkan aturan satu lencana per orang dan melatih staf bahwa kesopanan tidak menimbulkan kontrol akses. Varian terkait: menahan pintu tetap terbuka, meminjam lencana, atau menyamar sebagai kurir atau kontraktor.

## Daftar periksa tanda bahaya

Waspadai kombinasi dari sinyal-sinyal berikut:

- Desakan atau ancaman: "bertindak sekarang", "akun akan ditutup dalam 24 jam".
- Permintaan untuk melewati prosedur normal: mengubah detail rekening, mengirim kartu hadiah, berbagi kode MFA.
- Pengirim tak lazim: domain yang mirip (`micros0ft-support.example`), domain email gratis untuk eksekutif "internal", reply-to yang berbeda dari alamat From.
- Lampiran tak terduga atau tautan pendek — arahkan kursor untuk melihat tujuan sebenarnya sebelum mengklik.
- Meminta rahasia: kata sandi, kode sekali pakai, data pelanggan, atau pembayaran di luar alur persetujuan normal.
- Tekanan emosional: ketakutan, pujian, rasa penasaran, atau otoritas dari orang yang lebih senior.
- Kerahasiaan: "jangan beri tahu siapa pun, kesepakatan ini harus diam sampai Senin."
- Petunjuk teknis: halaman login yang dijangkau dari email, permintaan memasang alat akses jarak jauh, panggilan yang tepat datang saat Anda menerima prompt MFA.

## Membela faktor manusia

Teknologi menyaring sebagian besar sisanya; budaya dan prosedur menangani sisanya lagi.

### Budaya pelaporan

- Buat pelaporan mudah dan tanpa kambing hitam. Targetnya adalah melapor dalam hitungan menit, bukan hukuman karena mengklik.
- Rayakan laporan cepat: "email ini dilaporkan dalam 40 detik" lebih baik daripada mempermalukan pengguna yang tertipu oleh ujian.
- Publikasikan ringkasan singkat pasca-insiden agar semua orang tahu seperti apa penipuan yang sedang marak saat ini.
- Beri penghargaan pada perilaku yang Anda inginkan: orang yang mempertanyakan permintaan tak terduga justru sedang melindungi organisasi.

### Prosedur verifikasi

| Prosedur | Apa yang dicegahnya |
|---|---|
| Panggilan balik memakai nomor dari direktori resmi, bukan nomor yang ada di pesan | Vishing, pretexting |
| Persetujuan ganda untuk pembayaran dan perubahan detail rekening | BEC |
| Verifikasi tatap muka atau video untuk permintaan eksekutif yang "mendesak" | Penyamaran sebagai otoritas |
| Prosedur formal untuk akses fisik: satu lencana per orang, tantang orang asing | Tailgating |
| Jangan pernah membagikan kode MFA, meski dengan "dukungan TI" | Vishing dengan bypass MFA |
| Verifikasi tautan dengan mengetik alamatnya atau memakai bookmark, bukan tautan di email | Phishing kredensial |

### Dukungan teknis untuk manusia

Autentikasi email (SPF, DKIM, DMARC), banner pengirim eksternal, sandboxing tautan, dan MFA mengurangi volume yang masuk ke kotak masuk — tetapi perlakukan mereka sebagai lapisan di bawah prosedur di atas, bukan penggantinya.

## Skenario

Seorang akuntan menerima panggilan dari seseorang yang mengaku sebagai auditor eksternal:

1. Penelepon menyebut nomor penugasan yang nyata (diambil dari situs web perusahaan).
2. Ia meminta akuntan "mengonfirmasi paket dokumen" dengan masuk lewat tautan yang dikirim via SMS — sebuah langkah smishing.
3. Ketika akuntan ragu, penelepon bersikap tegas dan menyebut tenggat yang ditetapkan oleh CFO.
4. Tanggapan yang benar dari akuntan: tolak kanal tersebut, tutup telepon, hubungi kembali lewat nomor firma audit yang dipublikasikan, dan laporkan interaksi itu ke tim keamanan.

Tanpa malware, tanpa exploit — hanya skrip yang sudah dilatih. Panggilan balik yang diverifikasi merusaknya.

## Lihat juga

- [Lanskap Ancaman](threat-landscape.md)
- [Jenis Malware](malware.md)
- [Triad CIA](cia-triad.md)
- [Kata Sandi & Autentikasi](passwords-authentication.md)
