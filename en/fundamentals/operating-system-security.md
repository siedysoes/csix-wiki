# Operating System Security

Operating system security is the set of controls that protect a host — its users, files, processes, and network connections — from unauthorized access and compromise. Whether you are securing a laptop, a server, or a cloud instance, the same core principles apply: limit who can do what, keep the system patched, log what happens, and encrypt what matters.

## Least Privilege

Least privilege means every user, process, and service runs with the minimum permissions needed to do its job, and nothing more.

- Do not use an administrator account for daily work.
- Services should run as dedicated low-privilege accounts, not `root` or `SYSTEM`.
- Remove software and accounts that are no longer needed — every extra permission is an extra attack path.
- Grant elevated access just in time, not standing.

On Linux this is enforced with users, groups, and `sudo`; on Windows with User Account Control (UAC), group membership, and privilege rights. An attacker who compromises a non-admin process is far more limited than one who inherits admin rights.

## Patching

Unpatched systems are the most reliable entry point for attackers, and some flaws become wormable. Historical examples include MS17-010 (EternalBlue, used by WannaCry) and the Apache Log4j remote code execution flaw **CVE-2021-44228** (Log4Shell).

Practical patching guidance:

1. Maintain an accurate asset inventory — you cannot patch what you do not know about.
2. Subscribe to vendor security advisories for your OS and key software.
3. Define SLAs: critical/actively exploited flaws within days, routine updates on a monthly cycle.
4. Test before broad deployment, but do not let testing stall emergency patches.
5. Reboot when required; a kernel patch that has not been applied is not applied.

## Host Firewall

A host firewall protects a single machine even when the network perimeter fails.

```powershell
# Windows: block inbound SMB except from a management subnet
New-NetFirewallRule -DisplayName "Block SMB Inbound" -Direction Inbound `
  -Protocol TCP -LocalPort 445 -Action Block
```

```bash
# Linux (ufw): allow SSH and web only, deny the rest
sudo ufw default deny incoming
sudo ufw allow 22/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

Default-deny inbound, explicit allow for required services. The same idea appears in cloud security groups.

## Disk Encryption

Full-disk encryption protects data when a device is lost or stolen, because the disk cannot be read outside the machine without the key.

| OS | Tool | Typical key/unlock method |
|---|---|---|
| Windows Pro/Enterprise | BitLocker | TPM + PIN or recovery password |
| macOS | FileVault 2 | User password / recovery key |
| Linux | LUKS (cryptsetup) | Passphrase or keyfile |
| Mobile | Android FBE / iOS Data Protection | Device passcode-derived keys |

Important limitations:

- Encryption protects data **at rest**. It does nothing against malware running in an unlocked, logged-in session.
- Store recovery keys in a safe place (Entra ID/AD, password manager). An unrecoverable key is a self-inflicted outage.
- On Linux, LUKS protects the volume; `/home` on an unencrypted partition or swap may still leak unless separately encrypted.

## Windows vs Linux: Accounts and Permissions

| Aspect | Windows | Linux |
|---|---|---|
| Identity store | Local SAM, Active Directory domain | `/etc/passwd`, LDAP/SSSD |
| Superuser | `Administrator` / `SYSTEM` | `root` (or a user with sudo) |
| Permission model | ACLs on files/objects; NTFS | Owner/group/other bits plus ACLs (`setfacl`) |
| Elevation | UAC prompt, `runas` | `sudo` with configured `/etc/sudoers` |
| Service accounts | Virtual accounts (`NT SERVICE\...`) | Systemd units with `User=` |
| Auditing | Security event log with SACLs | `auditd` |

Useful everyday checks:

```bash
# Who has sudo rights?
sudo grep -E '^(root|%sudo)' /etc/sudoers
# Who is logged in and since when?
who
```

```powershell
# List local administrators
Get-LocalGroupMember -Group "Administrators"
```

## Logging and Audit

You cannot investigate what you never recorded.

- **Windows Event Viewer**: Security log (logon/logoff 4624/4625, privilege use, policy changes), System log, and PowerShell logs. Centralize with Windows Event Forwarding or a SIEM.
- **Linux journald**: `journalctl -u ssh -f` for live service logs; persistent storage requires `/var/log/journal`. For deeper syscall auditing use `auditd`.
- **Retention**: keep logs long enough to detect slow-moving incidents, synchronize time with NTP, and restrict write access so attackers cannot cover tracks.

```
$ journalctl -u ssh --since "2 hours ago" | grep -i failed
Nov 11 02:14:03 srv01 sshd[812]: Failed password for invalid user admin ...
```

Repeated failures like these are your signal for brute force — if anyone is looking.

## Secure Deletion Myths

Common misconceptions worth unlearning:

- **Deleting a file or emptying the recycle bin** only unlinks the pointer; the data remains until overwritten.
- **One overwrite pass is enough on modern SSDs.** For HDDs, a single full overwrite is generally sufficient; the old "seven-pass" DoD standard is unnecessary.
- **`shred` guarantees erasure.** On SSDs with wear leveling, journaled file systems, or RAID, software overwrite cannot guarantee every copy is destroyed. Free space is also invisible to the file system.
- **Encryption is the real answer.** With full-disk encryption, "secure deletion" becomes destroying the key — this is why encryption-based disposal is the recommended approach for SSDs.

For decommissioned media, use the vendor's secure-erase command (ATA/NVMe sanitize), destroy the key, or physically destroy the drive.

## Baseline Hardening with CIS Benchmarks

A **CIS Benchmark** is a consensus configuration standard from the Center for Internet Security describing hardened settings for a specific OS, application, or cloud service, organized into implementation groups.

Typical workflow:

1. Choose your asset (e.g., *CIS Microsoft Windows 11 Benchmark*).
2. Decide on a profile: Level 1 (defensible, minimal performance impact) vs Level 2 (higher security, more operational cost).
3. Scan with a tool (Microsoft Baseline Configuration Analyzer, OpenSCAP, Nessus, or your MDM).
4. Remediate, document exceptions with a business reason, and re-scan regularly.

Example: a Level 1 Linux rule may require `net.ipv4.conf.all.send_redirects = 0`. Failing that rule is a finding; disabling a service you actually need is an exception you own.

## See also

- [The CIA Triad](cia-triad.md)
- [Network Security Basics](network-basics.md)
- [Passwords & Authentication](passwords-authentication.md)
- [Malware Types](malware.md)
- [Study Roadmap](../career/study-roadmap.md)
