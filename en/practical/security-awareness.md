# Security Awareness Programs

A security awareness program teaches people to recognize and respond to security threats through repeated, practical education rather than a one-time slide deck. It matters because attackers target humans directly — phishing, pretexting, and business email compromise bypass even well-built technical controls.

## Why humans are targeted

Machines have patches; people have habits. Attackers go after humans because:

- One convincing email bypasses perimeter defenses entirely.
- Credentials handed over voluntarily are indistinguishable from legitimate logins in many logs.
- People are rushed, helpful, and authority-prone — exactly the traits social engineers exploit.
- A single click can start a ransomware chain or wire transfer fraud.

Technical controls still matter (see the fundamentals on social engineering), but the human layer is where the first decision is made.

Common pretexts used against staff:

| Pretext | Typical lure | Who it targets |
| --- | --- | --- |
| Invoice/payment change | "Update our bank details before paying this invoice" | Finance, accounting |
| Credential harvesting | "Your mailbox is full / password expires today" | Everyone |
| Whaling | CEO-style urgent wire request | Executives, assistants |
| IT support reset | "I need your MFA code to finish the migration" | Help desk, all staff |
| Document share | "Contract_FINAL.pdf shared with you" | Widespread blast radius |

A program should teach people to recognize the pattern — unexpected urgency plus a request to bypass process — not just to spot one specific logo or sender.

## Building a program

A sustainable program has four pillars:

1. **Leadership buy-in.** Executives must fund it, model it (leaders take the training too), and communicate why it matters. Without visible sponsorship, participation stalls and managers quietly exempt their teams.
2. **Baseline measurement.** Run an initial phishing simulation and a short knowledge survey before training, so you know where you stand and can show improvement later.
3. **Role-based training cadence.** Short, frequent beats long and rare: a 10-minute module monthly or quarterly for everyone, with deeper tracks for developers, finance (wire fraud), and executives (whaling). Include just-in-time nudges when someone fails a simulation.
4. **Positive reporting culture.** Make reporting frictionless (one-click report button, a dedicated address), celebrate reporters, and separate measurement from punishment.

## Phishing simulation rules of engagement

Simulations test behavior, not character. Written rules keep them ethical and effective:

- **Never punish.** No public shaming, no mandatory remedial assignments framed as penalty, no manager escalation for a single click. Punishment teaches people to hide mistakes, which destroys reporting.
- **Always educate.** Anyone who clicks immediately sees a short, friendly explanation of the red flags they missed — in the moment, while attention is high.
- **Disclose the program.** Tell staff that simulations happen (without revealing schedule or sender details). Deception about the program's existence erodes trust; deception about specific emails builds skill.
- **Stay realistic but safe.** Simulations may imitate brands and urgency, but must not collect real credentials into an uncontrolled system, must not link to malware, and must not target people in a vulnerable moment (e.g., right after a public incident).
- **Accessible design.** Provide a reporting path that works for everyone, and never penalize a user who forwards a message to the security team instead of clicking.
- **Governance.** Get HR and legal sign-off, define data retention for simulation results, and restrict who can see individual results.

## Metrics that matter

| Metric | Definition | Why it matters |
| --- | --- | --- |
| Click rate | % of recipients who clicked the simulated link | Quick baseline, but noisy and easy to game |
| Report rate | % of recipients who reported the simulation (or a real phishing) | Best leading indicator — reporting stops real attacks before analysts find them |
| Time-to-report | Median time from delivery to the first report | Measures detection speed; minutes vs. days decides blast radius |
| Repeat-click rate | % who click more than once over time | Identifies who needs support, not blame |
| Credential-entry rate | % who submitted data on the fake page | Distinguishes casual clicks from dangerous handoffs |
| Training completion | % who finish required modules | Compliance floor, not a security outcome |

**Why report rate matters more than click rate:** a zero-click campaign that nobody reports is still a success for the attacker — they learned the controls worked and nobody noticed. A 20% click rate with a 70% report rate means the organization detects and responds fast, which is what limits real damage. Optimize for people who speak up, not for people who never err.

## Handling suspected phishing as an end user

Give staff a simple, memorable procedure:

1. **Verify.** Check the sender address (not just the display name), hover links to see the real destination, and be suspicious of unexpected urgency, attachments, or requests to change payment details. When in doubt, contact the person or department through a known-good channel — never the number or address in the message.
2. **Report.** Use the report-phishing button or forward the message to the security team. Reporting a false alarm costs nothing; not reporting a real one costs a lot.
3. **Don't click, don't reply.** Do not open attachments, do not enter credentials, do not reply to the sender. Do not forward the raw message to coworkers "as a warning" — send it to the security team instead.
4. **If you already clicked.** Report immediately, change the affected password from a different device, and tell the security team which data you entered. Early disclosure is always treated as help, never as fault.

```text
Subject: Urgent: your mailbox will be closed in 24h
From:   it-support@examp1e-helpdesk.com   <- look-alike domain
Action: do NOT click; report with the button; verify via the internal directory
```

## Quarterly program calendar

| Quarter | Focus | Activities |
| --- | --- | --- |
| Q1 | Foundations & baseline | Leadership briefing, baseline phishing simulation, all-staff fundamentals module (phishing, passwords, MFA) |
| Q2 | Payments & impersonation | Wire/BEC simulation, finance deep-dive workshop, developer secure-coding session |
| Q3 | Physical & supply chain | Badge-tailgating spot checks, vendor/USB drop exercise, phishing re-test with report-rate goal |
| Q4 | Incident readiness & review | Ransomware tabletop exercise, annual results review with leadership, plan next year's budget and calendar |

Between quarters: monthly micro-training (5 minutes), continuous live-phishing reporting with fast feedback, and onboarding modules for new hires within their first week.

## Roles and responsibilities

| Role | Responsibility |
| --- | --- |
| Security team | Runs simulations, triages reports, produces metrics |
| Leadership | Sponsors the program, reviews quarterly results, models participation |
| Managers | Ensure team participation, reinforce a no-blame culture |
| HR / Legal | Approves rules of engagement, handles data privacy of results |
| All staff | Complete training, report suspicious messages, verify unusual requests |

## See also

- [Social Engineering](../fundamentals/social-engineering.md)
- [Incident Response Basics](incident-response.md)
- [Passwords & Authentication](../fundamentals/passwords-authentication.md)
- [Cybersecurity Learning Path](../career/learning-path.md)
