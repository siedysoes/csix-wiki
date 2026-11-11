# Secure Coding Practices

Secure coding means building software so that common vulnerabilities are prevented by design rather than patched after release. It matters because fixing a defect in production costs far more than preventing it during design, and because most real-world breaches trace back to a small set of well-known coding mistakes.

## Shift-left mindset

"Shift left" means moving security work earlier in the development lifecycle — requirements, design, and pull requests — instead of only testing at the end. Practical shift-left actions:

- Threat-model new features before writing code.
- Run static analysis (SAST) and dependency scans in CI, not after release.
- Review security requirements alongside functional requirements.
- Give developers fast feedback: a failing check on the same day it is introduced.

## Input validation

Treat every input as hostile: validate type, length, range, and format on the server side. Allow-lists (accept only known-good values) beat deny-lists (block known-bad values), which inevitably miss variants.

### Parameterized queries

Never build SQL by string concatenation. Use parameterized queries or prepared statements so user data is never parsed as code.

```python
# Vulnerable: attacker input becomes SQL
query = "SELECT * FROM users WHERE name = '" + name + "'"

# Safe: parameters are bound separately from the query structure
cursor.execute("SELECT * FROM users WHERE name = ?", (name,))
```

The same principle applies to LDAP queries, OS commands (use argument arrays, not shell strings), and path handling (canonicalize, then verify the resolved path stays inside the intended directory).

## Output encoding and XSS

Cross-site scripting happens when untrusted data is rendered without context-appropriate encoding. Encode for the specific output context:

| Context | Example | Defense |
| --- | --- | --- |
| HTML body | `<p>{comment}</p>` | HTML entity encoding, framework auto-escaping |
| HTML attribute | `<input value="{name}">` | Attribute encoding, quote the attribute |
| JavaScript | `var x = "{name}";` | JSON serialization, avoid inline scripts |
| URL | `<a href="/search?q={q}">` | Percent-encoding, validate scheme |
| CSS | `style="..."` | Avoid user-controlled styles entirely |

Framework defaults (React escaping, Django templates, Jinja auto-escape) cover the common cases — do not disable them, and use Content-Security-Policy as a second layer.

## Error handling without leaking secrets

Errors should be useful to users and safe for the system:

- Return generic messages to users ("Something went wrong, reference ID 1234").
- Log details server-side with the reference ID, never to the client.
- Never reveal stack traces, SQL fragments, internal IPs, or library versions in production responses.
- Fail closed: if an authorization check errors out, deny access rather than allow it.

```text
HTTP/1.1 500 Internal Server Error
{"error": "internal_error", "request_id": "b7c1f0"}

# Server log (not the client): full exception, query, user ID, stack trace
```

## Secrets management

Hardcoded credentials leak through source control, binaries, and logs. The rules:

- Never commit API keys, passwords, or private keys — add pre-commit scans (e.g., gitleaks, git-secrets) to catch mistakes.
- Load secrets at runtime from environment variables or a secret manager (HashiCorp Vault, AWS Secrets Manager, Azure Key Vault, SOPS for files in Git).
- Rotate secrets on a schedule and immediately after any suspected exposure.
- Scope each secret to the least privilege it needs; separate secrets per environment.
- Treat a secret that reached a commit history as compromised: rotate first, scrub history later.

## Dependency hygiene

Most projects inherit vulnerabilities through third-party libraries. Keep dependencies healthy:

- Maintain an **SBOM** (software bill of materials) in a standard format such as CycloneDX or SPDX, generated in CI.
- Automate updates with Dependabot, Renovate, or equivalent, and review changes like any other code.
- Pin versions (lockfiles), verify checksums, and prefer well-maintained packages with small dependency trees.
- Subscribe to advisory feeds (OSV, GitHub Advisory Database, NVD) for the packages you use.
- Remove unused dependencies; every unused package is unmonitored attack surface.

## Secure code review checklist

Work through this list on every review that touches security-relevant code:

1. **Authentication**: password storage (Argon2/bcrypt/scrypt), session invalidation, MFA paths.
2. **Authorization**: every request checked server-side; no IDOR — object access verified per user.
3. **Input**: validated server-side; allow-lists; no raw SQL/shell/path construction from input.
4. **Output**: encoded for context; CSP and security headers set.
5. **Errors**: generic to clients, detailed in logs, fail closed.
6. **Secrets**: none in code or config; loaded from a manager.
7. **Dependencies**: pinned, scanned, SBOM updated.
8. **Crypto**: modern algorithms, unique salts/IVs, no home-grown crypto, secure randomness (`secrets` module, `crypto.getRandomValues`).
9. **Concurrency/DoS**: rate limits, request size limits, timeouts on outbound calls.
10. **Logging**: security events logged without leaking PII or secrets.

## Mapping to OWASP ASVS and CWE

Standards turn the checklist into something auditable:

- **OWASP ASVS** (Application Security Verification Security Standard) provides tiered requirements (L1–L3) you can adopt as acceptance criteria per feature.
- **CWE** (Common Weakness Enumeration) categorizes the defects themselves — e.g., CWE-89 (SQL injection), CWE-79 (XSS), CWE-798 (hardcoded credentials).
- **OWASP Top 10** is the popular risk-oriented summary; link each finding to its CWE so trends can be tracked over time.

Example: a finding of "SQL injection in search endpoint" maps to OWASP A03:2021 Injection, ASVS V5.x, and CWE-89 — giving you risk context, a requirement to verify against, and a machine-readable identifier for your scanner.

## See also

- [Web Security & the OWASP Top 10](../fundamentals/web-security-owasp.md)
- [Cryptography Basics](../fundamentals/cryptography-basics.md)
- [Vulnerability Management](vulnerability-management.md)
- [Building a Home Lab](hands-on-lab.md)
