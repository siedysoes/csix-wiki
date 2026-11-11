# Cloud Security Basics

Cloud security is the practice of protecting data, applications, and infrastructure hosted in public and private clouds. Because cloud services run on shared infrastructure and are reachable over the internet by default, most cloud breaches come from misconfiguration and over-permissive access rather than novel attacks.

## The shared responsibility model

The provider secures the cloud; you secure what you put in it. Where the line falls depends on the service model — this is the single most common source of false confidence in cloud security.

| Area | IaaS (VMs, storage, network) | PaaS (managed platforms, serverless) | SaaS (email, CRM, collaboration) |
|---|---|---|---|
| Physical data centers and hardware | Provider | Provider | Provider |
| Network infrastructure and hypervisor | Provider | Provider | Provider |
| OS patching of the underlying host | Provider | Provider | Provider |
| Guest OS, runtime, middleware | Customer | Provider | Provider |
| Applications and code | Customer | Customer | Provider (you configure it) |
| Identity, access, and keys | Customer | Customer | Customer |
| Data classification and encryption | Customer | Customer | Customer |
| Network configuration (firewalls, security groups) | Customer | Customer (partly) | Customer (access settings) |

In every model you keep the same three jobs: who can get in (IAM), what they are allowed to do (permissions), and where the data goes (configuration and encryption).

## Top misconfigurations

| Misconfiguration | What goes wrong | First fix |
|---|---|---|
| Public object storage | Buckets containing exports, backups, or credentials become readable by anyone with the URL | Block public access at the account level, then review per-bucket policies |
| Over-permissive IAM | Users or services with `*` on everything; one compromise becomes full account takeover | Least-privilege policies, remove unused roles, review permissions on a schedule |
| No MFA | Password-only console and API access; a phished credential is enough | Enforce MFA for all human users, administrators first |
| Open security groups / firewall rules | `0.0.0.0/0` on SSH (22), RDP (3389), or a database port | Restrict to known ranges or a jump host; never expose databases |
| Long-lived access keys | Static keys committed to code or stored on laptops, valid for years, rarely rotated | Short-lived roles and workload identities; rotate and audit what remains |

Most of these are visible in a five-minute review of the account's storage, IAM, and network pages. That is precisely why they get missed — nobody assigns a five-minute review.

## IAM: least privilege and roles

Start from "no access" and add only what a task requires.

- **Roles for workloads, not keys.** Give VMs, containers, and CI pipelines IAM roles so the cloud issues short-lived credentials automatically instead of you embedding static keys.
- **Federated identity for humans.** Single sign-on from your identity provider with MFA; no local cloud accounts for people.
- **Scope every policy** by resource, action, and condition — source IP, required tag, MFA present.
- **Separate environments.** Different accounts or subscriptions for production, staging, and sandbox; never share admin credentials across them.
- **Break-glass admin.** One heavily monitored emergency account, disabled by default, used only when normal access fails.
- **Review on a schedule.** Quarterly permission reviews; delete roles and keys nobody can justify.

Access reviews can start from the provider's own CLI:

```bash
# What policies are attached to a workload role?
aws iam list-attached-role-policies --role-name app-role

# Is this bucket reachable by the public?
aws s3api get-bucket-policy-status --bucket company-exports

# Does the root account have MFA enabled? (1 = yes, 0 = no)
aws iam get-account-summary --query 'SummaryMap.AccountMFAEnabled'
```

An error from `get-bucket-policy-status` means the bucket has no policy — which is not the same as private. Public access can also come from an ACL or a shared access point, so check the account-level "block public access" setting as well.

## Cloud logging and audit trails

Every major provider records control-plane activity: who created, changed, or deleted what, when, and from which address. These activity logs (audit trails) are your primary evidence for both configuration drift and incident investigation.

- Enable the provider's audit log service (CloudTrail, Activity Log, Activity Tracker) in **every** region or subscription — logging is typically per-region and off by default.
- Stream logs to a separate account or storage location that workload administrators cannot delete; attackers commonly cover tracks by stopping logging first.
- Alert on high-signal events: logging disabled, a new IAM user or role created, a bucket policy made public, a security group opened to the world, MFA removed.
- Collect data-plane logs too (object access logs, database audit logs): control-plane logs show a resource was changed, not what was read out of it.

## Infrastructure as code and configuration drift

Defining infrastructure in code — Terraform, CloudFormation, Pulumi, Bicep — makes security review possible before anything exists: the policy lives in a pull request instead of a console click.

Configuration drift is what happens when reality moves away from the code: someone fixes production through the console, and the next `apply` silently reverts it or fails. Two controls keep this honest:

1. **Everything through the pipeline.** Console changes are exceptions, recorded and written back into code the same day.
2. **Drift detection.** Run the provider's drift detection or `terraform plan` on a schedule and treat any unexpected difference as a finding.

Scanning code before deploy catches issues that will exist everywhere; scanning the running account catches issues that already exist. You need both, because either one alone leaves a blind spot.

## Scenario: an exposed export bucket

An intern creates a bucket named `company-exports` to share a CSV with a contractor, sets it to public-read, and moves on. Nothing breaks, so nobody notices — until a scanner or an object access log flags it.

1. **Detect.** The CSPM scan or storage activity log shows a public grant, or `get-bucket-policy-status` returns `IsPublic: true`.
2. **Contain.** Turn on block public access at the account level (one setting, all buckets), then remove the specific ACL or policy.
3. **Assess.** Read the object access logs to see whether anything was downloaded. That answer decides whether this is an incident with notification duties or a near miss.
4. **Fix the cause.** Offer an approved sharing path — presigned URLs or a managed file-transfer service — so the workaround does not come back next month.
5. **Prevent.** Give the bucket an owner tag and add a policy check to the IaC pipeline so the same creation fails review next time.

## CSPM in one paragraph

Cloud Security Posture Management (CSPM) tools continuously read your cloud configuration and compare it against a baseline of good practice — provider hardening guides, CIS Benchmarks, and your own policy — then report (and sometimes auto-remediate) deviations such as public buckets, open security groups, or unencrypted disks. CSPM does not stop an attacker who is already inside a workload and does not fix vulnerable code; it shrinks the gap between intended and actual configuration, which is where most cloud breaches begin.

## Hardening checklist

- [ ] MFA enforced for all human users; no long-term access keys for people
- [ ] Root / break-glass account locked down, credentials stored offline, usage alerted
- [ ] Least-privilege roles for workloads; no static keys in code, images, or notebooks
- [ ] Object storage: account-level public access blocked, encryption enabled by default
- [ ] Security groups and firewalls: no `0.0.0.0/0` on management or database ports
- [ ] Audit logging enabled everywhere and shipped to a separate account, with alerts on log-disable events
- [ ] Encryption at rest on by default for disks, buckets, and databases; TLS enforced in transit
- [ ] Centralized identity provider with conditional access and an offboarding process
- [ ] Resource inventory with mandatory tagging — you cannot secure unknown assets
- [ ] IaC pipeline with policy checks and scheduled drift detection
- [ ] Vulnerability scanning for images and workloads, with a named owner per finding
- [ ] Backups outside the production account, tested with a real restore

## See also

- [The CIA Triad](../fundamentals/cia-triad.md)
- [Network Security Basics](../fundamentals/network-basics.md)
- [Passwords & Authentication](../fundamentals/passwords-authentication.md)
- [Vulnerability Management](vulnerability-management.md)
