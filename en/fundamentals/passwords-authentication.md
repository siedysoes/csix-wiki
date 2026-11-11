# Passwords & Authentication

Passwords remain the most common authentication method on the internet, and most account compromises still trace back to weak, reused, or phished credentials rather than to broken cryptography. This page covers how password strength actually works, how to store and verify them safely, and how modern multi-factor and passwordless options compare.

## Password Entropy: Length vs Complexity

Entropy measures how many possible combinations an attacker must guess, in bits. Roughly: `entropy = length × log2(alphabet size)`.

| Password | Length | Alphabet | Approx. entropy | Resistance to offline attack |
|---|---|---|---|---|
| `P@ssw0rd` | 8 | ~70 chars (if mixed+symbols forced) | ~49 bits | Weak — pattern is a common template |
| `Tr0ub4dor&3` | 11 | ~70 chars | ~64 bits | Moderate, but human-chosen patterns reduce it in practice |
| `correct horse battery staple` | 28 | 26 (lowercase + spaces) | ~131 bits | Strong — many independent words |

Takeaways:

- **Length dominates.** Longer passphrases beat short "complex" passwords.
- **Human-created complexity is predictable.** `Password1!` satisfies most old policies and is in every breach list.
- **Randomness beats memorability games.** A generator producing 16+ random characters is stronger than any memorable pattern.
- **Entropy only helps if the attacker cannot precompute it.** Breach dictionaries and rulesets (mangling `a` to `@`) collapse many "complex" passwords instantly.

## Password Managers

A password manager lets one strong master secret protect a vault of unique, randomly generated credentials.

Why it matters: the real risk is not a single weak password, it is **reuse**. One breached site yields credentials that are replayed against your email, bank, and work accounts (credential stuffing).

Practical guidance:

- Use a reputable manager with a strong, long master passphrase and never reuse that passphrase.
- Enable MFA on the vault itself — it is the highest-value target you own.
- Prefer managers that support autofill only on the matching domain, to blunt phishing of the vault.
- Enterprise alternatives exist (directory-managed SSO, managed browser profiles), but the principle is the same: unique credentials everywhere.

## Why Reuse Is Dangerous

```
Breach dump:  alice@example.com : Summer2024!
Attacker:     tries alice@example.com / Summer2024! on 50 popular services
Result:       3 accounts compromised, one of them the email that resets everything
```

Email is the crown jewel: password resets flow through it. Treat the mailbox as the root of trust for account recovery.

## How Servers Should Store Passwords

Never store plaintext, and never store a fast unsalted hash.

| Approach | Verdict |
|---|---|
| Plaintext | Unacceptable — one breach exposes everyone |
| Base64 / reversible encryption | Unacceptable — key management plus reversibility |
| MD5 / SHA-1 / SHA-256 (unsalted) | Unacceptable — rainbow tables and GPU-speed guessing |
| Salted SHA-256 | Better, but still too fast |
| bcrypt / scrypt / Argon2id | Correct — deliberately slow and memory-hard |

Modern recommended setup (NIST-aligned practice): **Argon2id** with a per-user random salt stored alongside the hash, or bcrypt with a work factor tuned so each verification takes roughly 100-500 ms on your hardware. Store the algorithm and parameters with the hash so you can upgrade later without resetting passwords.

Pepper (a secret key kept outside the database) can add another layer, but it must live in a secrets manager, not in source code.

## Multi-Factor Authentication, Ranked

| Method | Factor type | Phishing resistance | Approximate strength |
|---|---|---|---|
| SMS one-time codes | Something you have | Weak — relayed in real time, SIM-swap risk | Better than nothing |
| Email one-time codes | Something you have | Weak | Better than nothing |
| TOTP (authenticator app) | Something you have | Medium — codes can be entered on a fake site | Strong for most users |
| Push approval | Something you have | Medium — vulnerable to MFA fatigue spam | Strong with number matching |
| Hardware security key (FIDO2/U2F) | Something you have (with possession of private key) | Strong — origin-bound | Very strong |
| Passkeys (FIDO2 credential) | Something you have + biometric unlock | Strong — no shared secret to steal | Very strong |

Key points:

- SMS is the weakest common option (SS7 weaknesses, SIM swapping), but SMS-based MFA still blocks far more attacks than no MFA.
- TOTP shares a secret; if that secret leaks, codes can be forged. It is also defeated by real-time phishing proxies.
- FIDO2/WebAuthn credentials are bound to the legitimate domain, so a phishing site cannot use them at all.

## Passkeys in Brief

A passkey is a FIDO2 credential: a cryptographic key pair where the private key stays on your device (or in a synced password manager) and is unlocked by biometrics or a device PIN. The server only ever holds the public key.

- No password to phish, guess, or reuse.
- The browser checks the relying party ID, so the credential will not work on a look-alike domain.
- Device loss is handled by platform sync (iCloud Keychain, Google Password Manager, 1Password) or by a backup method.
- Migration matters: for high-risk accounts, prefer cross-device sync or registered backup keys so a lost phone does not lock you out.

## Account Recovery Pitfalls

Recovery is the back door attackers love, because it bypasses your carefully designed login.

- Security questions ("first pet", "mother's maiden name") are often publicly discoverable — treat them as usernames, not secrets.
- SMS recovery reintroduces SIM-swap risk even if login itself uses passkeys.
- Help-desk identity verification based on details available on social media is weak.
- Design recovery to require stronger evidence than login: known device, waiting periods, and human review for high-value changes.
- Document the process. Undocumented recovery flows are where consistency and auditability die.

## Password Policy Guidance (NIST SP 800-63B Style)

Modern guidance inverts most legacy rules:

| Do | Do not |
|---|---|
| Require at least 8 characters for all users, encourage longer (e.g., 64) | Force composition rules (upper/lower/digit/symbol) |
| Allow all printable characters and paste | Force periodic rotation on a fixed schedule (e.g., every 90 days) |
| Check new passwords against breach corpora (HIBP-style) | Block password managers or long passphrases |
| Rate-limit and add delay after repeated failures | Let users pick from known-bad lists |
| Offer MFA and passwordless options | Store or transmit passwords in reversible form |

Why no forced rotation? Regular mandated changes push users toward incremental, predictable passwords (`Summer2024!` → `Autumn2024!`). Rotate only on evidence of compromise, and screen continuously against breach lists instead.

## Scenario: Hardening a Login

Current state: 8-character minimum, composition rule, 90-day rotation, SMS reset, no rate limiting.

Target state:

```
min length 12 (support up to 64), any printable characters
screen candidate against known-breached password corpus -> reject + explain
10 attempts / 15 min per account, then exponential backoff + alert
MFA: passkey preferred, TOTP fallback, SMS only as last resort
recovery: registered device + email confirmation, 24h hold on new payout details
hashing: Argon2id (m=64MiB, t=3, p=4) with per-user salt
```

This change removes the rules that annoyed users while removing the paths attackers actually use.

## See also

- [Cryptography Basics](cryptography-basics.md)
- [Web Security & the OWASP Top 10](web-security-owasp.md)
- [Social Engineering](social-engineering.md)
- [Operating System Security](operating-system-security.md)
- [Cybersecurity Learning Path](../career/learning-path.md)
