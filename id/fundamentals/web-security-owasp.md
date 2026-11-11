# Keamanan Web & OWASP Top 10

OWASP Top 10 adalah dokumen kesadaran yang diperbarui berkala, memuat risiko keamanan aplikasi web paling kritis. Dokumen ini menjadi kosakata bersama dalam pelatihan, code review, dan persyaratan keamanan, dan edisi 2021 mengelompokkan risiko berdasarkan akar penyebab, bukan gejala per gejala.

## Ikhtisar OWASP Top 10 (2021)

| ID | Kategori | Artinya |
|---|---|---|
| A01 | Broken Access Control | Pengguna dapat bertindak di luar izin yang semestinya (melihat, mengubah, menghapus data orang lain) |
| A02 | Cryptographic Failures | Data sensitif terekspos karena enkripsi yang hilang atau lemah |
| A03 | Injection | Data tidak tepercaya ditafsirkan sebagai kode (SQL, perintah OS, LDAP) |
| A04 | Insecure Design | Pola kontrol yang hilang atau lemah pada tahap desain |
| A05 | Security Misconfiguration | Kredensial default, error yang terlalu detail, cloud bucket terbuka, fitur tidak perlu |
| A06 | Vulnerable and Outdated Components | Menggunakan library/kerangka dengan cacat yang diketahui |
| A07 | Identification and Authentication Failures | Logika login, sesi, atau pemulihan kredensial yang lemah |
| A08 | Software and Data Integrity Failures | Mempercaya update, plugin, atau artefak CI/CD tanpa verifikasi |
| A09 | Security Logging and Monitoring Failures | Insiden tidak terdeteksi karena tidak ada yang dicatat atau dipantau |
| A10 | Server-Side Request Forgery (SSRF) | Server dibujuk untuk mengirim request ke sumber daya internal |

Perhatikan pergeseran dari 2017: injection (sebelumnya A01) turun ke A03 sementara access control dan insecure design naik, mencerminkan asal-usul breach nyata.

## Rincian Kasus Utama

### Injection dan SQL Injection

Injection terjadi setiap kali input tidak tepercaya digabungkan ke dalam sebuah interpreter. Dalam SQL:

```python
# Rentan: input menjadi bagian dari sintaks query
query = "SELECT * FROM users WHERE username = '" + username + "'"
# Input:  ' OR '1'='1' --
# Result: SELECT * FROM users WHERE username = '' OR '1'='1' --'
```

Perbaikannya adalah **parameterized query** (statement yang sudah dikompilasi sebelumnya), yang memisahkan data dan kode:

```python
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

Keluarga injection lainnya: OS command injection (`; rm -rf /`), LDAP injection, XML/XXE, template injection, dan NoSQL injection.

### Cross-Site Scripting (XSS)

XSS memungkinkan penyerang menjalankan JavaScript di browser pengguna lain, mencuri sesi atau menulis ulang halaman.

- **Stored XSS**: payload jahat disimpan (misalnya, komentar forum) dan disajikan ke setiap penonton.
- **Reflected XSS**: payload di URL dipantulkan kembali dalam respons.
- **DOM-based XSS**: script sisi klien memproses `location.hash` atau sejenisnya secara tidak aman.

```html
<!-- Unsafe -->
<div>Hello, <%= request.getParameter("name") %></div>

<!-- Safe: context-aware output encoding -->
<div>Hello, <%= HtmlEncoder.encode(name) %></div>
```

Pertahanan berlapis (defense in depth): output encoding sesuai konteks, Content Security Policy yang ketat, dan cookie `HttpOnly` agar script yang dicuri tidak bisa membaca token sesi.

### Cross-Site Request Forgery (CSRF)

CSRF menyalahgunakan fakta bahwa browser secara otomatis melampirkan cookie. Jika Anda sudah login ke bank dan mengunjungi halaman jahat, halaman tersebut dapat memicu request yang mengubah state menggunakan sesi Anda.

Pertahanan: **token anti-CSRF** (nilai per-sesi yang tidak bisa ditebak dan tidak bisa dibaca penyerang), cookie `SameSite=Lax/Strict`, dan mewajibkan autentikasi ulang untuk aksi sensitif. Metode HTTP yang aman (GET) tidak boleh pernah mengubah state.

### Autentikasi Rusak dan IDOR

**Broken authentication** mencakup credential stuffing, invalidasi sesi yang lemah, token reset yang bisa ditebak, serta izin password seperti `123456`.

**Insecure Direct Object Reference (IDOR)** adalah bug kontrol akses di mana mengubah sebuah pengenal (identifier) membuka catatan orang lain:

```
GET /api/invoices/10041   -> faktur Anda   (OK)
GET /api/invoices/10042   -> faktur pengguna lain (IDOR)
```

Server harus memeriksa *otorisasi pada setiap request*, bukan hanya apakah pengguna sudah login. Itulah mengapa IDOR berada di bawah A01 Broken Access Control.

## Dasar Pengembangan Aman

| Praktik | Mencegah apa | Titik awal |
|---|---|---|
| Validasi input (allow-list) | Injection, data cacat | Validasi tipe, panjang, rentang, format di server |
| Parameterized query | Injection SQL/NoSQL | ORM atau prepared statement di mana pun |
| Output encoding | XSS | Auto-escaping framework plus encoding sesuai konteks |
| Security header | Clickjacking, XSS, downgrade | `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Strict-Transport-Security` |
| Pemindaian dependensi | A06 komponen rentan | SCA di CI (misalnya `npm audit`, Dependabot, OWASP Dependency-Check) | 
| Manajemen rahasia | Kebocoran kredensial | Environment/secret store, jangan pernah di Git |
| Logging terpusat | A09 kegagalan deteksi | Catat event autentikasi, penolakan akses, aksi admin dengan request ID |

Contoh header respons untuk aplikasi web yang diperkuat:

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: no-referrer
```

Kesadaran akan kerentanan yang diketahui juga penting: Log4Shell (**CVE-2021-44228**), cacat remote code execution di Log4j, menunjukkan bagaimana satu dependensi transitif dapat menempatkan ribuan sistem dalam risiko — itulah mengapa diperlukan software bill of materials (SBOM) dan proses patch yang cepat.

## Skenario Tinjauan Singkat

```
$ curl -i "https://shop.example/api/order?id=7"
HTTP/1.1 200 OK
Set-Cookie: session=abc123; Path=/
Content-Type: application/json

{"id":7,"total":49.90,"card":"**** 4242"}
```

Temuan: ID dirujuk langsung tanpa pemeriksaan kepemilikan (IDOR), cookie sesi tidak memiliki atribut `HttpOnly`/`Secure`/`SameSite`, dan metadata kartu lengkap dikembalikan padahal hanya empat digit terakhir yang diperlukan (kegagalan data minimization). Perbaiki otorisasi di sisi server terlebih dahulu — filter dan penyembunyian UI tidak dihitung.

## Lihat juga

- [Dasar Keamanan Jaringan](network-basics.md)
- [Dasar Kriptografi](cryptography-basics.md)
- [Password & Autentikasi](passwords-authentication.md)
- [Keamanan Sistem Operasi](operating-system-security.md)
- [Jalur Belajar Keamanan Siber](../career/learning-path.md)
