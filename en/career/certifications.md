# Security Certifications Guide

Certifications signal knowledge to employers, and some contracts or regulations list them explicitly, but they cost money and time, so pick the one that matches the role you want. Requirements and pricing change often — as of writing, treat everything below as high-level orientation and verify current details and pricing on the official website before you commit.

## Comparison table

| Certification | Level | Focus | Prerequisites / experience (high level) | Who it suits |
|---------------|-------|-------|------------------------------------------|--------------|
| CompTIA Security+ | Entry | Defensive / vendor-neutral fundamentals | No formal prerequisite; assumes basic IT and networking knowledge | Career changers and juniors who need a widely recognized baseline; frequently listed in job requirements |
| eJPT | Entry | Offensive | No formal prerequisite, but solid networking, OS and tool knowledge is expected; hands-on practical exam | Beginners who want a practical pentest credential instead of a multiple-choice one |
| Google Cybersecurity Certificate | Entry | Defensive / fundamentals | No formal prerequisite | People starting from zero who want a structured introduction before self-study |
| CompTIA CySA+ | Intermediate | Defensive (threat detection and analysis) | Builds on entry-level knowledge; many employers expect Security+ or equivalent experience | SOC analysts moving from alert triage into detection and analysis |
| PNPT | Intermediate | Offensive | Practical exam; assumes fundamentals plus genuine hands-on practice | Pentesters who want a practical, report-focused assessment |
| CEH | Intermediate | Offensive (knowledge and terminology) | Training or relevant experience expected; routes vary, so verify on the official site | Candidates who need a widely recognized credential for HR screening or compliance-driven roles |
| GCIH | Intermediate | Defensive (incident handling) | Designed for practitioners; no strict formal prerequisite, but hands-on experience is expected | Incident responders and SOC analysts formalizing their skills |
| OSCP | Advanced | Offensive | Hands-on practical exam; expects strong networking, Linux/Windows and scripting foundations | Pentesters who want a demanding, practical credential |
| OSWE | Advanced / expert | Offensive (web application, white-box code review) | Expects OSCP-level discipline plus solid code review skills | Web application specialists |
| CISSP | Expert | Management and broad security architecture | 5 years paid work experience in 2+ domains, with a one-year waiver (for example an approved degree or credential) | Experienced practitioners moving into architecture, management or leadership |
| CISM | Expert | Management (governance and programs) | Requires substantial relevant experience including management-level work | Managers responsible for security programs and risk decisions |

## Well-known requirements explained

- **CISSP**: the classic experience-first credential — five years of cumulative paid work experience across two or more of its domains, with one year waivable through an approved degree or certification. Without the experience you may hold an associate-level recognition; check the issuing body's current rules.
- **OSCP**: an entirely hands-on practical exam — you exploit systems in a timed lab setting rather than answering multiple choice. It rewards scripting, enumeration and persistence more than memorization.
- **Security+**: vendor-neutral entry level with no formal prerequisite and broad coverage of security concepts; it proves baseline knowledge, not job readiness by itself.
- **Practical vs. knowledge-based**: eJPT, PNPT, OSCP and OSWE are hands-on exams, while CySA+, CEH, GCIH, CISSP and CISM are primarily written, proctored exams. Employers usually weight practical exams more heavily for technical roles.
- **eJPT and PNPT**: both target early-career pentesters and are assessed practically; exam format and time limits have changed before, so check the official site for the current rules.
- **GCIH and CySA+**: both assume you already work with logs or incidents; they certify existing practice rather than replacing lab time.

## Vendor-neutral vs. vendor-specific

- The credentials above are mostly vendor-neutral: they test concepts that transfer between employers.
- Vendor-specific training (cloud providers, SIEM or EDR platforms) becomes useful once you know which tools your target job ads mention; take it after the fundamentals, not before.
- Job ads are the source of truth: collect a dozen ads for your target role and count which certifications are listed as required versus preferred.

## Costs and maintenance

- The exam fee is rarely the full cost: some routes include mandatory training, and retakes are usually paid separately.
- Several credentials recertify on a cycle with continuing-education obligations — factor the ongoing time and cost into your choice before buying.
- Prices differ by country, currency and delivery method, so verify current details and pricing on the official website rather than quoting forum figures.
- Student, military and bundle discounts exist in some regions; check the official site before paying the list price.

## Red flags to avoid

- Any provider promising a guaranteed pass, a certificate without an exam, or a "lifetime" credential with no verification.
- Buying the most expensive credential first on the assumption it makes the others unnecessary.
- Certifications that never appear in job ads for your target region and role.
- Assuming CISSP or CISM qualify you for management purely through the exam, ignoring the experience requirement.
- Budgeting for a first attempt only: retakes are usually paid separately, so plan for that possibility before scheduling.

## How to choose

Start with three questions: do you enjoy investigating patterns or breaking into systems? Do you prefer writing reports, code, or policy documents? And what first role exists in the job market of the country you want to work in?

| If you... | Start with |
|-----------|-----------|
| Enjoy debugging, pattern hunting and incident stories | Blue team: Security+, then CySA+ |
| Enjoy breaking things and writing clear reports | Red team: eJPT, then PNPT |
| Care about code quality and web applications | AppSec: a baseline cert, then the web-focused practical track |
| Like policy, risk and talking to stakeholders | GRC: Security+ now, CISSP or CISM after you have the experience |
| Are starting from complete zero | Google Cybersecurity Certificate, then the learning path stages |

### Blue team (defensive) path

Sequence: Security+ to fill gaps → CySA+ → GCIH. Build evidence alongside the exams: SIEM lab time, log analysis, detection rules and incident write-ups. Add cloud or vendor-specific credentials later based on the job ads you are targeting, not on forum advice alone.

### Red team (offensive) path

Sequence: eJPT → PNPT → OSCP → OSWE. This is a practical chain: each exam should follow real lab hours, and employers will ask for write-ups, not just the letters. Networking and scripting carry more weight here than any single certificate.

### Management / GRC path

Sequence: Security+ (to close technical gaps) → CISSP or CISM. These are experience-first credentials — the exam is the smaller part of the commitment, and the experience requirement means you plan this several years into your career.

### Spending rules

- Read 10-20 job descriptions for the role you want; note which certs are requirements and which are merely nice-to-have.
- One cert at a time: build the skill, prove it in a lab or project, then sit the exam.
- Never buy a credential you cannot demonstrate in an interview.
- Check renewal and continuing-education obligations before paying, since several of these recertify on a cycle.

## Evidence alongside certificates

A certificate gets you past an automated screen; demonstration gets you through the interview. Pair each credential with something you built: a hardening checklist after a baseline cert, a detection lab after an analytics cert, or published machine write-ups after a practical exam. If you cannot show the skill in a five-minute walkthrough, the certificate alone will not carry you.

- Keep write-ups short enough that a hiring manager will read them, but detailed enough that a peer could repeat your steps.
- Link your portfolio from your CV and check that the links still work after every platform change.
- Refresh the portfolio after each completed lab — a stale portfolio is worse than none.

## Notes for international readers

- Requirements vary by country: some governments and regulated industries list specific certifications as mandatory, while many private-sector ads treat them as optional.
- Job titles differ across regions — "SOC analyst", "security operations analyst" and "information security analyst" may describe similar work; read the duties, not just the label.
- Most of these exams are available in English, and some offer other language versions; check the issuing body's current options, exam rules and pricing on the official website.

## See also

- [The CIA Triad](../fundamentals/cia-triad.md)
- [Network Security Basics](../fundamentals/network-basics.md)
- [Cybersecurity Learning Path](learning-path.md)
- [Study Roadmap](study-roadmap.md)
