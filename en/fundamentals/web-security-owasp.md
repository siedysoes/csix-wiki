# Web Security & the OWASP Top 10

The OWASP Top 10 is a periodically updated awareness document listing the most critical web application security risks. It is the common vocabulary used in training, code reviews, and security requirements, and the 2021 edition groups risks by root cause rather than individual symptom.

## OWASP Top 10 (2021) Overview

| ID | Category | What it means |
|---|---|---|
| A01 | Broken Access Control | Users can act outside their intended permissions (view, edit, delete others' data) |
| A02 | Cryptographic Failures | Sensitive data exposed through missing or weak encryption |
| A03 | Injection | Untrusted data interpreted as code (SQL, OS commands, LDAP) |
| A04 | Insecure Design | Missing or weak control patterns at the design stage |
| A05 | Security Misconfiguration | Default credentials, verbose errors, open cloud buckets, unnecessary features |
| A06 | Vulnerable and Outdated Components | Using libraries/frameworks with known flaws |
| A07 | Identification and Authentication Failures | Weak login, session, or credential recovery logic |
| A08 | Software and Data Integrity Failures | Trusting updates, plugins, or CI/CD artifacts without verification |
| A09 | Security Logging and Monitoring Failures | Incidents go undetected because nothing is logged or watched |
| A10 | Server-Side Request Forgery (SSRF) | The server is tricked into making requests to internal resources |

Note the shift from 2017: injection (formerly A01) dropped to A03 as access control and insecure design rose, reflecting where real breaches originate.

## The Big Ones in Depth

### Injection and SQL Injection

Injection happens whenever untrusted input is concatenated into an interpreter. In SQL:

```python
# Vulnerable: input becomes part of the query syntax
query = "SELECT * FROM users WHERE username = '" + username + "'"
# Input:  ' OR '1'='1' --
# Result: SELECT * FROM users WHERE username = '' OR '1'='1' --'
```

The fix is **parameterized queries** (precompiled statements), which keep data and code separate:

```python
cursor.execute("SELECT * FROM users WHERE username = %s", (username,))
```

Other injection families: OS command injection (`; rm -rf /`), LDAP injection, XML/XXE, template injection, and NoSQL injection.

### Cross-Site Scripting (XSS)

XSS lets an attacker run JavaScript in another user's browser, stealing sessions or rewriting pages.

- **Stored XSS**: malicious payload saved (e.g., a forum comment) and served to every viewer.
- **Reflected XSS**: payload in the URL is echoed back in the response.
- **DOM-based XSS**: the client-side script unsafely processes `location.hash` or similar.

```html
<!-- Unsafe -->
<div>Hello, <%= request.getParameter("name") %></div>

<!-- Safe: context-aware output encoding -->
<div>Hello, <%= HtmlEncoder.encode(name) %></div>
```

Defense in depth: output encoding by context, a strict Content Security Policy, and `HttpOnly` cookies so stolen script cannot read session tokens.

### Cross-Site Request Forgery (CSRF)

CSRF abuses the fact that the browser automatically attaches cookies. If you are logged into your bank and visit a malicious page, that page can trigger a state-changing request using your session.

Defenses: **anti-CSRF tokens** (unpredictable per-session values the attacker cannot read), `SameSite=Lax/Strict` cookies, and requiring re-authentication for sensitive actions. Safe HTTP methods (GET) must never change state.

### Broken Authentication and IDOR

**Broken authentication** covers credential stuffing, weak session invalidation, predictable reset tokens, and allowing passwords like `123456`.

**Insecure Direct Object Reference (IDOR)** is an access-control bug where changing an identifier reveals someone else's record:

```
GET /api/invoices/10041   -> your invoice   (OK)
GET /api/invoices/10042   -> another user's invoice (IDOR)
```

The server must check *authorization on every request*, not just whether the user is logged in. This is why IDOR sits under A01 Broken Access Control.

## Secure Development Basics

| Practice | What it prevents | Starting point |
|---|---|---|
| Input validation (allow-list) | Injection, malformed data | Validate type, length, range, format on the server |
| Parameterized queries | SQL/NoSQL injection | ORM or prepared statements everywhere |
| Output encoding | XSS | Framework auto-escaping plus context-aware encoding |
| Security headers | Clickjacking, XSS, downgrade | `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Strict-Transport-Security` |
| Dependency scanning | A06 vulnerable components | SCA in CI (e.g., `npm audit`, Dependabot, OWASP Dependency-Check) |
| Secrets management | Credential leakage | Environment/secret store, never in Git |
| Centralized logging | A09 detection failures | Log auth events, access denials, admin actions with request IDs |

Example response headers for a hardened web app:

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: no-referrer
```

Known-vulnerability awareness matters too: Log4Shell (**CVE-2021-44228**), a remote code execution flaw in Log4j, showed how a single transitive dependency can put thousands of systems at risk — hence the need for a software bill of materials (SBOM) and rapid patch processes.

## A Quick Review Scenario

```
$ curl -i "https://shop.example/api/order?id=7"
HTTP/1.1 200 OK
Set-Cookie: session=abc123; Path=/
Content-Type: application/json

{"id":7,"total":49.90,"card":"**** 4242"}
```

Findings: the ID is directly referenced with no ownership check (IDOR), the session cookie lacks `HttpOnly`/`Secure`/`SameSite` attributes, and full card metadata is returned when only the last four digits are needed (data minimization failure). Fix authorization server-side first — filters and UI hiding do not count.

## See also

- [Network Security Basics](network-basics.md)
- [Cryptography Basics](cryptography-basics.md)
- [Passwords & Authentication](passwords-authentication.md)
- [Operating System Security](operating-system-security.md)
- [Cybersecurity Learning Path](../career/learning-path.md)
