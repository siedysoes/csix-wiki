# Social Engineering

Social engineering is the manipulation of people into performing actions or disclosing information that weaken security, rather than attacking technology directly. It works because it exploits normal human traits — trust, urgency, helpfulness — so technical controls alone cannot stop it.

## The main techniques

| Technique | Channel | Core trick | Typical goal |
|---|---|---|---|
| Phishing | Mass email | Fake but plausible message with a link or attachment | Steal credentials or install malware |
| Spear phishing | Targeted email | Personalized to one person using researched details | Compromise a specific high-value account |
| Vishing | Phone call | Caller poses as IT, bank, or executive | Extract codes, passwords, or payments |
| Smishing | SMS / messaging apps | Short message with urgent link or callback number | Credential theft or malware install |
| Business email compromise (BEC) | Email | Impersonate executive, vendor, or partner | Redirect payments to attacker accounts |
| Pretexting | Any, usually voice or email | Invented scenario where the attacker has a "reason" | Obtain records, access, or sensitive data |
| Baiting | Physical or digital | Tempting lure (free USB, "leaked" file) | Get victim to run malware or visit a trap |
| Quid pro quo | Phone / in person | "I'll help you if you give me..." | Gain credentials or physical access |
| Tailgating | Physical | Following an authorized person through a door | Enter restricted areas without a badge |

## Phishing family in detail

**Phishing** is broad: thousands of identical emails claiming a package is undelivered or a password will expire. It works on volume — one percent clicking is enough.

**Spear phishing** narrows to one target. An attacker reads a company's press releases and LinkedIn profiles, then emails the finance manager referencing a real deal. Success rates are far higher because nothing looks generic.

**Vishing** moves to voice, where call-back verification is harder. A classic variant is a fake "IT support" call on a busy Monday: "We're seeing errors on your account, can you read me the code we just texted you?" The attacker, who triggered the real MFA prompt, receives the code.

**Smishing** is the same idea over SMS, often impersonating a delivery service or bank with a shortened link.

**BEC** usually involves no malware at all. The attacker studies invoice formats, then sends a clean-looking email:

```text
From: cfo@your-company.example        (displayed name: CFO's real name)
To: accounts.payable@your-company.example
Subject: Re: Invoice #4471 - urgent, supplier bank changed

Hi team,
Please update our supplier's bank details before today's run —
the account change was confirmed on the call this morning.
New IBAN: XX00 1234 5678 9012 3456
Send the confirmation reply to this thread only.
Thanks,
CFO
```

The reply-to trick, the urgency, and the "don't discuss it elsewhere" instruction are all standard BEC markers. In 2020's Twitter breach, social engineering of staff — not a software flaw — helped attackers take over high-profile accounts.

## Pretexting, baiting, and quid pro quo

- **Pretexting** is a fabricated context that makes the request feel routine. An attacker calls pretending to be from HR collecting an updated tax form, or from the landlord asking for an apartment key. The key skill is research: the more detail the pretext contains, the less it is questioned.
- **Baiting** offers something tempting. Physically, an infected USB labelled "Payroll Q4" left in the parking lot; digitally, a file named `salary_adjustment.xlsx` on a shared drive. Curiosity does the rest.
- **Quid pro quo** trades a favor: "We're the security team doing a password audit — read me your current password and we'll upgrade you to the new scheme." Unlike vishing's fake emergency, it pretends to be a service.

## Physical techniques

**Tailgating** (or piggybacking) is following an employee through a badge-locked door, often with arms full of boxes and a friendly "could you get that?". **Locking out** tailgating means enforcing badge-per-person and training staff that politeness does not override access control. Related variants: propping doors open, borrowing a badge, or posing as a delivery driver or contractor.

## Red flags checklist

Watch for combinations of these signals:

- Urgency or threat: "act now", "account will be closed within 24 hours".
- Requests to bypass normal process: change bank details, send gift cards, share MFA codes.
- Unusual sender: look-alike domains (`micros0ft-support.example`), free mail domains for "internal" execs, reply-to differing from the From address.
- Unexpected attachments or shortened links — hover to see the real destination before clicking.
- Asks for secrets: passwords, one-time codes, customer data, or payments outside the normal approval flow.
- Emotional pressure: fear, flattery, curiosity, or authority from a senior person.
- Secrecy: "don't tell anyone, this deal must stay quiet until Monday."
- Technical tells: login pages reached from an email, requests to install remote-access tools, calls that arrive exactly when you receive an MFA prompt.

## Defending the human factor

Technology filters most of the rest; culture and procedures handle the remainder.

### Report culture

- Make reporting easy and blameless. The goal is minutes-to-report, not punishment for clicking.
- Celebrate fast reports: "this email was reported in 40 seconds" beats shaming users who fell for a test.
- Publish short post-incident summaries so everyone learns what the current scam looks like.
- Reward the behavior you want: people who challenge unexpected requests are protecting the organization.

### Verification procedures

| Procedure | What it prevents |
|---|---|
| Callback using a number from the official directory, never the one in the message | Vishing, pretexting |
| Dual approval for payments and bank-detail changes | BEC |
| In-person or video verification for "urgent" executive requests | Impersonation of authority |
| Formal process for physical access: badge per person, challenge strangers | Tailgating |
| Never share MFA codes, even with "IT support" | MFA-bypass vishing |
| Verify links by typing the address or using a bookmark, not the email's link | Credential phishing |

### Technical support for humans

Email authentication (SPF, DKIM, DMARC), external-sender banners, link sandboxing, and MFA reduce the volume reaching inboxes — but treat them as layers beneath the procedures above, not replacements for them.

## Scenario

An accountant receives a call from someone claiming to be the external auditor:

1. The caller references a real engagement number (researched from the company website).
2. They ask the accountant to "confirm a document package" by logging in via a link sent by SMS — a smishing step.
3. When the accountant hesitates, the caller becomes authoritative and mentions a deadline set by the CFO.
4. The accountants' correct response: decline the channel, hang up, call back on the published audit-firm number, and report the interaction to security.

No malware, no exploit — just a rehearsed script. The verified callback breaks it.

## See also

- [Threat Landscape](threat-landscape.md)
- [Malware Types](malware.md)
- [The CIA Triad](cia-triad.md)
- [Passwords & Authentication](passwords-authentication.md)
