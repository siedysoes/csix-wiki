# Cryptography Basics

Cryptography is the science of encoding information so that only intended parties can read or modify it. It underpins nearly every security control you will use — TLS, password storage, code signing, VPNs — so knowing what each primitive does, and just as importantly what it does not do, is essential.

## Hashing vs Encryption vs Encoding

These three are constantly confused. They solve different problems.

| Property | Hashing | Encryption | Encoding |
|---|---|---|---|
| Reversible | No | Yes, with key | Yes, always |
| Key needed | None | Yes | None |
| Purpose | Integrity, fingerprints | Confidentiality | Representation (base64, URL, UTF-8) |
| Example | SHA-256, bcrypt | AES-GCM, ChaCha20 | Base64, hex, HTML entities |
| Typical use | File integrity, password storage | Data at rest, TLS | Data transport formats |

Key facts:

- **Hashing is one-way by design.** You cannot "decrypt" a hash; you can only try inputs until one matches.
- **Encoding is not security.** Base64 is trivially reversible — treating it as encryption is a common bug.
- **Encryption needs a key and a mode.** AES alone is not a decision; AES-GCM vs AES-CBC vs AES-ECB are very different.

## Symmetric vs Asymmetric Encryption

| Aspect | Symmetric | Asymmetric (public key) |
|---|---|---|
| Keys | One shared secret key | Key pair: public + private |
| Speed | Fast (orders of magnitude) | Slow, computationally expensive |
| Key distribution | Hard problem: how do you share the key? | Solved: publish the public key freely |
| Common algorithms | AES-128/256, ChaCha20 | RSA, ECC (Ed25519, P-256) |
| Typical use | Bulk data encryption | Key exchange, signatures, small payloads |

Real systems combine both. TLS uses asymmetric cryptography once to agree on a session key, then switches to symmetric encryption for the actual data — because symmetric is far faster.

### Digital signatures

A signature is asymmetric encryption (or equivalent) of a hash with the private key. Anyone with the public key can verify it. Signatures give you **authentication** and **non-replication**, not confidentiality.

## The TLS Handshake in Plain Language

TLS is what puts the padlock in the browser. A simplified modern (TLS 1.3) flow:

1. Client hello: "Here are the cipher suites I support, my random value, and the domain I want."
2. Server hello: "Here is my certificate, my chosen cipher, and my random value."
3. Client verifies the certificate chain against trusted root CAs, checks the domain matches and that it is not expired or revoked.
4. Both sides derive a shared session key (using ephemeral key exchange so future traffic stays secret even if the private key is later stolen — forward secrecy).
5. Application data flows, authenticated and encrypted with the symmetric session key.

If any step fails — wrong name, expired cert, untrusted issuer — the browser shows a warning. Users clicking through certificate warnings is a real and recurring attack enabler.

## Digital Certificates and PKI

A certificate binds a public key to an identity (a domain name, an organization, a code-signing identity). Trust comes from a chain:

```
Root CA (trusted by your OS/browser)
  └── Intermediate CA
        └── Leaf certificate for www.example.com
```

- **Certificate Authority (CA):** issues and signs certificates.
- **Registration Authority (RA):** validates the requester's identity before issuance.
- **Certificate Revocation:** via CRL (list of revoked serial numbers) or OCSP (real-time status query). Short-lived certificates reduce reliance on revocation checks.
- **Trust store:** the list of root CAs your system trusts. A compromised or maliciously added root CA can intercept any "secure" connection — hence the value of certificate pinning and transparency logs.

## Common Cryptographic Misuse

| Misuse | Why it is bad | Do this instead |
|---|---|---|
| Rolling your own crypto | Subtle flaws are invisible to you and obvious to attackers | Use vetted libraries (libsodium, OpenSSL, platform APIs) |
| AES-ECB mode | Identical plaintext blocks map to identical ciphertext; patterns leak | AES-GCM or ChaCha20-Poly1305 (authenticated encryption) |
| MD5 / SHA-1 for passwords | Fast by design; brute-forceable at enormous scale | bcrypt, scrypt, Argon2id |
| MD5/SHA-1 for signatures | Collision attacks are practical (SHAttered, 2017) | SHA-256 or SHA-3 |
| Static IV / reused nonce | Breaks GCM and CBC catastrophically | Generate a fresh random nonce per encryption |
| No integrity check | Enables padding-oracle and bit-flipping attacks | Use AEAD modes that authenticate ciphertext |
| Hard-coded secrets in repos | Trivially found by scanners | Secrets manager, rotate on exposure |

Rule: never invent an algorithm, a mode, a padding scheme, or a protocol. Compose well-known primitives from a maintained library.

## Password Storage: Salting, bcrypt, Argon2

Passwords must be stored as something you cannot reverse. The evolution:

1. **Plain text** — unacceptable; one breach exposes everyone.
2. **Unsalted fast hash (MD5/SHA-1)** — defeated by rainbow tables and GPUs.
3. **Salted fast hash (SHA-256)** — kills rainbow tables, but still fast to brute-force.
4. **Slow, memory-hard KDFs** — bcrypt, scrypt, Argon2id — deliberately expensive so each guess costs the attacker real time and memory.

| Algorithm | Memory-hard | Tunable cost | Notes |
|---|---|---|---|
| bcrypt | No | Yes (work factor) | Widely available, well understood |
| scrypt | Yes | Yes | Good, less commonly default |
| Argon2id | Yes | Yes (time, memory, parallelism) | Winner of the Password Hashing Competition; current best practice |

Conceptual example (illustrative, not a runnable snippet):

```
password:  "correct horse battery staple"
salt:      9f2b... (random, unique per user, stored alongside hash)
params:    Argon2id, m=64MB, t=3, p=4
result:    $argon2id$v=19$m=65536,t=3,p=4$9f2b...$Kx7...
```

The salt is not secret. Its only job is to ensure two users with the same password get different hashes and to defeat precomputed tables.

## Scenario: Choosing a Cipher

You need to encrypt user uploads at rest:

- Wrong: `AES/ECB/PKCS5Padding` with a key from `config.properties` in Git.
- Right: AES-256-GCM, random 96-bit nonce per object, key in a cloud KMS with restricted IAM policy, nonce stored with the ciphertext, annual key rotation, and an audit log of key use.

The second option is not more code — it is a handful of correct choices from a library.

## See also

- [Network Security Basics](network-basics.md)
- [Passwords & Authentication](passwords-authentication.md)
- [Web Security & the OWASP Top 10](web-security-owasp.md)
- [The CIA Triad](cia-triad.md)
- [Security Certifications Guide](../career/certifications.md)
