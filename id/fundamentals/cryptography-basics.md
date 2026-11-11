# Dasar Kriptografi

Kriptografi adalah ilmu mengkode informasi agar hanya pihak yang dituju yang dapat membaca atau memodifikasinya. Hampir setiap kontrol keamanan yang akan Anda gunakan berdiri di atasnya — TLS, penyimpanan password, code signing, VPN — jadi mengetahui apa yang dilakukan setiap primitive, dan sama pentingnya apa yang tidak dilakukannya, adalah hal yang esensial.

## Hashing vs Enkripsi vs Encoding

Ketiganya sering tertukar. Ketiganya menyelesaikan masalah yang berbeda.

| Properti | Hashing | Enkripsi | Encoding |
|---|---|---|---|
| Dapat dibalik | Tidak | Ya, dengan kunci | Ya, selalu |
| Perlu kunci | Tidak | Ya | Tidak |
| Tujuan | Integritas, sidik jari | Kerahasiaan | Representasi (base64, URL, UTF-8) |
| Contoh | SHA-256, bcrypt | AES-GCM, ChaCha20 | Base64, hex, entity HTML |
| Penggunaan umum | Integritas file, penyimpanan password | Data at rest, TLS | Format transport data |

Fakta kunci:

- **Hashing bersifat satu arah (one-way) secara desain.** Anda tidak bisa "mendekripsi" hash; Anda hanya bisa mencoba input sampai ada yang cocok.
- **Encoding bukan keamanan.** Base64 mudah dibalik — memperlakukannya sebagai enkripsi adalah bug yang umum.
- **Enkripsi memerlukan kunci dan mode.** AES saja bukan sebuah keputusan; AES-GCM vs AES-CBC vs AES-ECB sangat berbeda.

## Enkripsi Simetris vs Asimetris

| Aspek | Simetris | Asimetris (kunci publik) |
|---|---|---|
| Kunci | Satu kunci rahasia bersama | Pasangan kunci: publik + privat |
| Kecepatan | Cepat (berbeda beberapa order magnitudo) | Lambat, secara komputasi mahal |
| Distribusi kunci | Masalah sulit: bagaimana Anda membagikan kunci? | Terpecahkan: publikasikan kunci publik dengan bebas |
| Algoritma umum | AES-128/256, ChaCha20 | RSA, ECC (Ed25519, P-256) |
| Penggunaan umum | Enkripsi data massal (bulk) | Pertukaran kunci, tanda tangan, payload kecil |

Sistem nyata menggabungkan keduanya. TLS menggunakan kriptografi asimetris sekali untuk menyepakati kunci sesi, lalu beralih ke enkripsi simetris untuk data yang sesungguhnya — karena simetris jauh lebih cepat.

### Tanda tangan digital

Tanda tangan adalah enkripsi asimetris (atau setara) terhadap hash dengan kunci privat. Siapa pun yang memiliki kunci publik dapat memverifikasinya. Tanda tangan memberi Anda **autentikasi** dan **non-repudiasi**, bukan kerahasiaan.

## Handshake TLS dengan Bahasa Sederhana

TLS adalah yang membuat gembok muncul di browser. Alur modern yang disederhanakan (TLS 1.3):

1. Client hello: "Berikut cipher suite yang saya dukung, nilai acak saya, dan domain yang saya tuju."
2. Server hello: "Berikut sertifikat saya, cipher pilihan saya, dan nilai acak saya."
3. Klien memverifikasi rantai sertifikat terhadap CA root tepercaya, memeriksa kecocokan domain, serta apakah sertifikat masih berlaku dan tidak dicabut.
4. Kedua pihak menurunkan kunci sesi bersama (menggunakan pertukaran kunci singkat/ephemeral agar trafik masa depan tetap rahasia walau kunci privat dicuri kemudian — forward secrecy).
5. Data aplikasi mengalir, diautentikasi dan dienkripsi dengan kunci sesi simetris.

Jika salah satu langkah gagal — nama salah, sertifikat kedaluwarsa, penerbit tidak tepercaya — browser menampilkan peringatan. Pengguna yang mengklik lewat peringatan sertifikat adalah pemicu serangan yang nyata dan berulang.

## Sertifikat Digital dan PKI

Sertifikat mengikat kunci publik ke sebuah identitas (nama domain, organisasi, identitas code-signing). Kepercayaan berasal dari sebuah rantai:

```
Root CA (dipercaya OS/browser Anda)
  └── Intermediate CA
        └── Leaf certificate untuk www.example.com
```

- **Certificate Authority (CA):** menerbitkan dan menandatangani sertifikat.
- **Registration Authority (RA):** memvalidasi identitas pemohon sebelum penerbitan.
- **Pencabutan sertifikat:** melalui CRL (daftar nomor seri yang dicabut) atau OCSP (query status waktu nyata). Sertifikat berumur pendek mengurangi ketergantungan pada pemeriksaan pencabutan.
- **Trust store:** daftar CA root yang dipercaya sistem Anda. CA root yang disusupi atau ditambahkan dengan jahat dapat menyadap koneksi "aman" mana pun — itulah mengapa certificate pinning dan transparency log memiliki nilai penting.

## Penyalahgunaan Kriptografi yang Umum

| Penyalahgunaan | Mengapa buruk | Lakukan ini sebagai gantinya |
|---|---|---|
| Merancang kripto sendiri | Cacat halus tidak terlihat oleh Anda, tetapi jelas bagi penyerang | Gunakan library yang sudah ditelaah (libsodium, OpenSSL, API platform) |
| Mode AES-ECB | Blok plaintext identik menghasilkan ciphertext identik; pola bocor | AES-GCM atau ChaCha20-Poly1305 (authenticated encryption) |
| MD5 / SHA-1 untuk password | Cepat secara desain; bisa di-brute-force dalam skala raksasa | bcrypt, scrypt, Argon2id |
| MD5/SHA-1 untuk tanda tangan | Serangan collision praktis (SHAttered, 2017) | SHA-256 atau SHA-3 |
| IV statis / nonce dipakai ulang | Merusak GCM dan CBC secara fatal | Hasilkan nonce acak baru untuk setiap enkripsi |
| Tanpa pemeriksaan integritas | Memungkinkan serangan padding-oracle dan bit-flipping | Gunakan mode AEAD yang mengautentikasi ciphertext |
| Rahasia hard-coded di repo | Ditemukan scanner dengan mudah | Secrets manager, rotasi saat terjadi kebocoran |

Aturan: jangan pernah menciptakan algoritma, mode, skema padding, atau protokol sendiri. Susun primitive yang sudah dikenal dari sebuah library yang terawat.

## Penyimpanan Password: Salting, bcrypt, Argon2

Password harus disimpan dalam bentuk yang tidak bisa dibalik. Evolusinya:

1. **Plain text** — tidak dapat diterima; satu breach membuka semuanya.
2. **Hash cepat tanpa salt (MD5/SHA-1)** — dikalahkan rainbow table dan GPU.
3. **Hash cepat dengan salt (SHA-256)** — menggagalkan rainbow table, tetapi masih cepat di-brute-force.
4. **KDF lambat dan memory-hard** — bcrypt, scrypt, Argon2id — sengaja mahal sehingga setiap tebakan menghabiskan waktu dan memori nyata bagi penyerang.

| Algoritma | Memory-hard | Biaya dapat disetel | Catatan |
|---|---|---|---|
| bcrypt | Tidak | Ya (work factor) | Umum tersedia, dipahami baik |
| scrypt | Ya | Ya | Baik, jarang jadi default |
| Argon2id | Ya | Ya (waktu, memori, paralelisme) | Pemenang Password Hashing Competition; praktik terbaik saat ini |

Contoh konseptual (ilustratif, bukan potongan kode yang bisa dijalankan):

```
password:  "correct horse battery staple"
salt:      9f2b... (acak, unik per pengguna, disimpan bersama hash)
params:    Argon2id, m=64MB, t=3, p=4
result:    $argon2id$v=19$m=65536,t=3,p=4$9f2b...$Kx7...
```

Salt bukan rahasia. Satu-satunya tugasnya memastikan dua pengguna dengan password sama mendapatkan hash berbeda, dan menggagalkan tabel pra-hitung.

## Skenario: Memilih Cipher

Anda perlu mengenkripsi unggahan pengguna saat disimpan (at rest):

- Salah: `AES/ECB/PKCS5Padding` dengan kunci dari `config.properties` di Git.
- Benar: AES-256-GCM, nonce acak 96-bit per objek, kunci di cloud KMS dengan kebijakan IAM terbatas, nonce disimpan bersama ciphertext, rotasi kunci tahunan, dan log audit penggunaan kunci.

Pilihan kedua tidak menambah kode — itu hanya sejumlah keputusan yang benar dari sebuah library.

## Lihat juga

- [Dasar Keamanan Jaringan](network-basics.md)
- [Password & Autentikasi](passwords-authentication.md)
- [Keamanan Web & OWASP Top 10](web-security-owasp.md)
- [Triad CIA](cia-triad.md)
- [Panduan Sertifikasi Keamanan](../career/certifications.md)
