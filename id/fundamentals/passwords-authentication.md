# Password & Autentikasi

Password masih merupakan metode autentikasi paling umum di internet, dan sebagian besar kompromi akun masih berpangkal pada kredensial yang lemah, dipakai ulang, atau didapat lewat phishing, bukan pada kriptografi yang rusak. Halaman ini menjelaskan sebenarnya bagaimana kekuatan password bekerja, cara menyimpan dan memverifikasinya dengan aman, serta perbandingan opsi multi-factor dan passwordless modern.

## Entropi Password: Panjang vs Kompleksitas

Entropi mengukur berapa banyak kombinasi yang harus ditebak penyerang, dalam bit. Kira-kira: `entropy = length × log2(alphabet size)`.

| Password | Panjang | Alfabet | Entropi kira-kira | Ketahanan terhadap serangan offline |
|---|---|---|---|---|
| `P@ssw0rd` | 8 | ~70 karakter (bila paksa campur+simbol) | ~49 bit | Lemah — polanya adalah template yang umum |
| `Tr0ub4dor&3` | 11 | ~70 karakter | ~64 bit | Sedang, tetapi pola pilihan manusia menurunkannya dalam praktik |
| `correct horse battery staple` | 28 | 26 (huruf kecil + spasi) | ~131 bit | Kuat — banyak kata independen |

Poin penting:

- **Panjanglah yang menentukan.** Passphrase panjang mengalahkan password pendek yang "kompleks".
- **Kompleksitas buatan manusia bisa ditebak.** `Password1!` memenuhi sebagian besar kebijakan lama dan ada di setiap daftar breach.
- **Kebetulan (randomness) mengalahkan permainan daya ingat.** Generator yang menghasilkan 16+ karakter acak lebih kuat daripada pola mudah diingat mana pun.
- **Entropi hanya berguna jika penyerang tidak bisa menghitungnya di awal.** Kamus breach dan ruleset (mengubah `a` menjadi `@`) langsung menjatuhkan banyak password "kompleks".

## Password Manager

Password manager memungkinkan satu rahasia master yang kuat melindungi brankas kredensial unik yang dihasilkan secara acak.

Mengapa ini penting: risiko yang sesungguhnya bukan satu password yang lemah, melainkan **reuse (pemakaian ulang)**. Satu situs yang membocorkan kredensial menghasilkan kredensial yang diputar ulang ke akun email, bank, dan pekerjaan Anda (credential stuffing).

Panduan praktis:

- Gunakan manager yang bereputasi dengan master passphrase panjang dan kuat, dan jangan pernah memakai ulang passphrase itu.
- Aktifkan MFA pada vault itu sendiri — itu sasaran bernilai tertinggi yang Anda miliki.
- Utamakan manager yang hanya mengisi otomatis pada domain yang cocok, untuk meredam phishing terhadap vault.
- Alternatif enterprise tersedia (SSO yang dikelola direktori, profil browser terkelola), tetapi prinsipnya sama: kredensial unik di mana pun.

## Mengapa Reuse Berbahaya

```
Breach dump:  alice@example.com : Summer2024!
Attacker:     mencoba alice@example.com / Summer2024! di 50 layanan populer
Result:       3 akun terkompromi, satu di antaranya email yang mereset semuanya
```

Email adalah harta paling berharga: reset password mengalir lewat sana. Perlakukan kotak masuk sebagai root of trust untuk pemulihan akun.

## Cara Server Seharusnya Menyimpan Password

Jangan pernah menyimpan plaintext, dan jangan pernah menyimpan hash cepat tanpa salt.

| Pendekatan | Penilaian |
|---|---|
| Plaintext | Tidak dapat diterima — satu breach membuka semuanya |
| Base64 / enkripsi yang bisa dibalik | Tidak dapat diterima — manajemen kunci plus kemampuan dibalik |
| MD5 / SHA-1 / SHA-256 (tanpa salt) | Tidak dapat diterima — rainbow table dan tebakan secepat GPU |
| SHA-256 dengan salt | Lebih baik, tetapi masih terlalu cepat |
| bcrypt / scrypt / Argon2id | Benar — sengaja lambat dan memory-hard |

Pengaturan modern yang direkomendasikan (praktik selaras NIST): **Argon2id** dengan salt acak per pengguna yang disimpan bersama hash, atau bcrypt dengan work factor yang disetel agar setiap verifikasi memakan waktu sekitar 100-500 ms di hardware Anda. Simpan algoritma dan parameter bersama hash agar Anda bisa meningkatkannya nanti tanpa mereset password.

Pepper (kunci rahasia yang disimpan di luar basis data) bisa menambah lapisan lain, tetapi harus berada di secrets manager, bukan di source code.

## Multi-Factor Authentication, Peringkat

| Metode | Jenis faktor | Ketahanan phishing | Kekuatan kira-kira |
|---|---|---|---|
| Kode sekali pakai via SMS | Sesuatu yang Anda miliki | Lemah — diteruskan secara real time, risiko SIM-swap | Lebih baik daripada tidak ada |
| Kode sekali pakai via email | Sesuatu yang Anda miliki | Lemah | Lebih baik daripada tidak ada |
| TOTP (aplikasi authenticator) | Sesuatu yang Anda miliki | Sedang — kode bisa dimasukkan di situs palsu | Kuat untuk sebagian besar pengguna |
| Persetujuan push | Sesuatu yang Anda miliki | Sedang — rentan terhadap spam MFA fatigue | Kuat dengan number matching |
| Kunci keamanan perangkat keras (FIDO2/U2F) | Sesuatu yang Anda miliki (dengan penguasaan kunci privat) | Kuat — terikat pada origin | Sangat kuat |
| Passkey (kredensial FIDO2) | Sesuatu yang Anda miliki + pembukaan biometrik | Kuat — tidak ada rahasia bersama yang bisa dicuri | Sangat kuat |

Poin kunci:

- SMS adalah opsi umum yang paling lemah (kelemahan SS7, SIM swapping), tetapi MFA berbasis SMS masih memblokir jauh lebih banyak serangan dibanding tanpa MFA.
- TOTP membagi sebuah rahasia; jika rahasia itu bocor, kode bisa dipalsukan. Proxy phishing waktu nyata juga bisa mengalahkannya.
- Kredensial FIDO2/WebAuthn terikat pada domain yang sah, sehingga situs phishing tidak bisa menggunakannya sama sekali.

## Passkey Secara Singkat

Passkey adalah kredensial FIDO2: pasangan kunci kriptografi di mana kunci privat tetap di perangkat Anda (atau di password manager yang tersinkron) dan dibuka dengan biometrik atau PIN perangkat. Server hanya menyimpan kunci publik.

- Tidak ada password yang bisa di-phishing, ditebak, atau dipakai ulang.
- Browser memeriksa relying party ID, sehingga kredensial tidak akan berfungsi di domain yang mirip.
- Kehilangan perangkat ditangani oleh sinkronisasi platform (iCloud Keychain, Google Password Manager, 1Password) atau metode cadangan.
- Migrasi itu penting: untuk akun berisiko tinggi, utamakan sinkronisasi lintas perangkat atau kunci cadangan yang terdaftar agar ponsel yang hilang tidak mengunci Anda keluar.

## Jebakan Pemulihan Akun

Pemulihan adalah pintu belakang yang disukai penyerang, karena melewati login yang Anda rancang dengan hati-hati.

- Pertanyaan keamanan ("hewan peliharaan pertama", "nama gadis ibu") sering bisa ditemukan secara publik — perlakukan sebagai username, bukan rahasia.
- Pemulihan via SMS memperkenalkan kembali risiko SIM-swap bahkan jika login sendiri menggunakan passkey.
- Verifikasi identitas help desk berdasarkan detail yang tersedia di media sosial adalah lemah.
- Rancang pemulihan agar memerlukan bukti yang lebih kuat daripada login: perangkat yang dikenal, masa tunggu, dan peninjauan manusia untuk perubahan bernilai besar.
- Dokumentasikan prosesnya. Alur pemulihan tanpa dokumentasi adalah tempat konsistensi dan auditabilitas mati.

## Panduan Kebijakan Password (Gaya NIST SP 800-63B)

Panduan modern membalik sebagian besar aturan warisan:

| Lakukan | Jangan |
|---|---|
| Wajibkan minimal 8 karakter untuk semua pengguna, dorong yang lebih panjang (misalnya 64) | Paksa aturan komposisi (huruf besar/kecil/angka/simbol) |
| Izinkan semua karakter yang dapat dicetak dan tempel | Paksa rotasi berkala dengan jadwal tetap (misalnya setiap 90 hari) |
| Periksa password baru terhadap korpus breach (gaya HIBP) | Blokir password manager atau passphrase panjang |
| Batasi laju dan tambahkan jeda setelah kegagalan berulang | Biarkan pengguna memilih dari daftar password yang sudah terbukti buruk |
| Tawarkan opsi MFA dan passwordless | Simpan atau transmisikan password dalam bentuk yang bisa dibalik |

Mengapa tidak paksa rotasi? Perubahan berkala yang dipaksakan mendorong pengguna ke password bertahap dan bisa ditebak (`Summer2024!` → `Autumn2024!`). Rotasi hanya atas adanya bukti kompromi, dan lakukan penyaringan terus-menerus terhadap daftar breach sebagai gantinya.

## Skenario: Memperkuat Login

Kondisi saat ini: minimal 8 karakter, aturan komposisi, rotasi 90 hari, reset via SMS, tanpa pembatasan laju.

Kondisi target:

```
min length 12 (support up to 64), any printable characters
screen candidate against known-breached password corpus -> reject + explain
10 attempts / 15 min per account, then exponential backoff + alert
MFA: passkey preferred, TOTP fallback, SMS only as last resort
recovery: registered device + email confirmation, 24h hold on new payout details
hashing: Argon2id (m=64MiB, t=3, p=4) with per-user salt
```

Perubahan ini menghapus aturan yang menyebalkan bagi pengguna sekaligus menghapus jalur-jalur yang benar-benar dipakai penyerang.

## Lihat juga

- [Dasar Kriptografi](cryptography-basics.md)
- [Keamanan Web & OWASP Top 10](web-security-owasp.md)
- [Social Engineering](social-engineering.md)
- [Keamanan Sistem Operasi](operating-system-security.md)
- [Jalur Belajar Keamanan Siber](../career/learning-path.md)
