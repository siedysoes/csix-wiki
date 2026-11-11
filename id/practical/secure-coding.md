# Praktik Secure Coding

Secure coding berarti membangun software sedemikian rupa sehingga kerentanan umum dicegah sejak dari desain, bukan ditambal setelah rilis. Ini penting karena memperbaiki cacat di produksi jauh lebih mahal daripada mencegahnya sejak tahap desain, dan karena sebagian besar pelanggaran nyata berakar pada sedikit sekali kesalahan coding yang sudah terkenal.

## Mindset shift-left

"Shift-left" berarti memindahkan pekerjaan keamanan lebih awal dalam siklus pengembangan — requirements, desain, dan pull request — bukan hanya diuji di akhir. Tindakan shift-left yang praktis:

- Lakukan threat modeling pada fitur baru sebelum menulis kode.
- Jalankan static analysis (SAST) dan pemindaian dependensi di CI, bukan setelah rilis.
- Tinjau requirement keamanan bersamaan dengan requirement fungsional.
- Berikan umpan balik cepat kepada developer: sebuah pemeriksaan yang gagal pada hari yang sama saat masalah diperkenalkan.

## Input validation

Perlakukan setiap input sebagai berbahaya: validasi tipe, panjang, rentang, dan format di sisi server. Allow-list (hanya menerima nilai yang sudah dikenal baik) lebih baik daripada deny-list (memblokir nilai yang sudah dikenal buruk), yang pasti melewatkan variasi-variasi baru.

### Parameterized query

Jangan pernah membangun SQL dengan concatenation string. Gunakan parameterized query atau prepared statement agar data pengguna tidak pernah diurai sebagai kode.

```python
# Vulnerable: attacker input becomes SQL
query = "SELECT * FROM users WHERE name = '" + name + "'"

# Safe: parameters are bound separately from the query structure
cursor.execute("SELECT * FROM users WHERE name = ?", (name,))
```

Prinsip yang sama berlaku untuk kueri LDAP, perintah OS (gunakan array argumen, bukan string shell), dan penanganan path (kanonikalisasi dulu, lalu verifikasi bahwa path hasil resolusi tetap berada di dalam direktori yang diinginkan).

## Output encoding dan XSS

Cross-site scripting (XSS) terjadi ketika data yang tidak tepercaya dirender tanpa encoding yang sesuai dengan konteksnya. Lakukan encoding untuk konteks output tertentu:

| Konteks | Contoh | Pertahanan |
| --- | --- | --- |
| Body HTML | `<p>{comment}</p>` | Encoding HTML entity, auto-escaping framework |
| Atribut HTML | `<input value="{name}">` | Encoding atribut, beri tanda kutip pada atribut |
| JavaScript | `var x = "{name}";` | Serialisasi JSON, hindari script inline |
| URL | `<a href="/search?q={q}">` | Percent-encoding, validasi skema |
| CSS | `style="..."` | Hindari sepenuhnya gaya yang dikendalikan pengguna |

Default framework (escaping React, template Django, auto-escape Jinja) sudah menangani kasus-kasus umum — jangan menonaktifkannya, dan gunakan Content-Security-Policy sebagai lapisan kedua.

## Penanganan error tanpa membocorkan rahasia

Error seharusnya berguna bagi pengguna dan aman bagi sistem:

- Kembalikan pesan generik kepada pengguna ("Terjadi kesalahan, referensi ID 1234").
- Catat detail di sisi server beserta referensi ID, jangan pernah ke klien.
- Jangan pernah menampilkan stack trace, potongan SQL, IP internal, atau versi library dalam respons produksi.
- Gagal ke mode tertutup: jika pemeriksaan otorisasi mengalami error, tolak akses alih-alih mengizinkannya.

```text
HTTP/1.1 500 Internal Server Error
{"error": "internal_error", "request_id": "b7c1f0"}

# Server log (not the client): full exception, query, user ID, stack trace
```

## Secrets management

Kredensial yang di-hardcode bocor melalui source control, binary, dan log. Aturannya:

- Jangan pernah commit API key, kata sandi, atau private key — tambahkan pemindaian pre-commit (misalnya gitleaks, git-secrets) untuk menangkap kesalahan semacam ini.
- Muat rahasia saat runtime dari environment variable atau secret manager (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, SOPS untuk file di Git).
- Rotasi rahasia secara berkala dan segera setelah ada dugaan kebocoran.
- Batasi setiap rahasia pada hak akses paling minim yang ia butuhkan; pisahkan rahasia per lingkungan.
- Perlakukan rahasia yang sudah masuk ke riwayat commit sebagai sudah terkompromikan: rotasi dulu, bersihkan riwayat kemudian.

## Kebersihan dependensi

Sebagian besar proyek mewarisi kerentanan melalui pustaka pihak ketiga. Jaga kesehatan dependensi:

- Pertahankan **SBOM** (software bill of materials) dalam format standar seperti CycloneDX atau SPDX, yang dihasilkan di CI.
- Otomatiskan pembaruan dengan Dependabot, Renovate, atau setara, dan tinjau perubahannya seperti kode lainnya.
- Kunci versi (lockfile), verifikasi checksum, dan utamakan paket yang terawat baik dengan pohon dependensi kecil.
- Berlangganan umpan advisory (OSV, GitHub Advisory Database, NVD) untuk paket yang Anda gunakan.
- Hapus dependensi yang tidak terpakai; setiap paket yang tidak digunakan adalah permukaan serangan yang tidak terpantau.

## Checklist review kode aman

Telusuri daftar berikut pada setiap review yang menyentuh kode yang relevan dengan keamanan:

1. **Authentication**: penyimpanan kata sandi (Argon2/bcrypt/scrypt), invalidasi sesi, jalur MFA.
2. **Authorization**: setiap request diperiksa di sisi server; tanpa IDOR — akses objek diverifikasi per pengguna.
3. **Input**: divalidasi di sisi server; allow-list; tanpa pembentukan SQL/shell/path mentah dari input.
4. **Output**: di-encode sesuai konteks; CSP dan security header sudah diatur.
5. **Errors**: generik untuk klien, detail di log, gagal ke mode tertutup.
6. **Secrets**: tidak ada di kode atau konfigurasi; dimuat dari secret manager.
7. **Dependencies**: dikunci versinya, dipindai, SBOM diperbarui.
8. **Crypto**: algoritma modern, salt/IV unik, tanpa kriptografi buatan sendiri, randomness yang aman (modul `secrets`, `crypto.getRandomValues`).
9. **Concurrency/DoS**: rate limit, batas ukuran request, timeout pada panggilan keluar.
10. **Logging**: peristiwa keamanan dicatat tanpa membocorkan PII atau rahasia.

## Pemetaan ke OWASP ASVS dan CWE

Standar mengubah checklist menjadi sesuatu yang bisa diaudit:

- **OWASP ASVS** (Application Security Verification Security Standard) menyediakan requirement berjenjang (L1–L3) yang bisa Anda adopsi sebagai kriteria penerimaan per fitur.
- **CWE** (Common Weakness Enumeration) mengategorikan cacat itu sendiri — misalnya CWE-89 (SQL injection), CWE-79 (XSS), CWE-798 (kredensial hardcoded).
- **OWASP Top 10** adalah ringkasan populer yang berorientasi pada risiko; tautkan setiap temuan ke CWE-nya agar tren dapat dilacak dari waktu ke waktu.

Contoh: temuan "SQL injection di endpoint pencarian" dipetakan ke OWASP A03:2021 Injection, ASVS V5.x, dan CWE-89 — memberi Anda konteks risiko, requirement untuk diverifikasi, serta identifikator yang dapat dibaca mesin untuk pemindai Anda.

## Lihat juga

- [Keamanan Web & OWASP Top 10](../fundamentals/web-security-owasp.md)
- [Dasar Kriptografi](../fundamentals/cryptography-basics.md)
- [Vulnerability Management](vulnerability-management.md)
- [Membangun Home Lab](hands-on-lab.md)
