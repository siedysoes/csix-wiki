# Digital Forensics Basics

Digital forensics is the science of identifying, preserving, collecting, and analyzing digital evidence so it can stand up to scrutiny. It matters because an incident response decision ("who took the data?") is only as good as the evidence behind it, and mishandled evidence can break both the investigation and any legal proceeding.

## Forensics vs. incident response

Both disciplines overlap, but they have different goals and tempos.

| Aspect | Incident Response | Digital Forensics |
| --- | --- | --- |
| Primary goal | Contain and recover quickly | Preserve and prove what happened |
| Time pressure | High (minutes to hours) | Lower (accuracy over speed) |
| Typical question | "Is the attacker still inside?" | "How did the file leave the system?" |
| Evidence handling | Capture first, document later | Document before touching anything |
| Output | Containment actions, IR report | Timeline, artifact report, court-ready evidence |

In practice you do both: grab volatile evidence first (order of volatility), then move to the slower, methodical forensic phase.

## Chain of custody

Chain of custody is the unbroken record of who handled evidence, when, and why. If any link is missing, the evidence can be challenged and thrown out.

For each piece of evidence, record:

- Unique evidence ID and description (asset tag, serial number).
- Date, time, and location of collection.
- Who collected it, who handled it next (each transfer signed).
- Storage conditions and any access since collection.
- Hash values (SHA-256) at collection and before every analysis session.

Store original evidence read-only; always work on a verified copy.

## Order of volatility (recap)

Collect the most volatile data first, because it disappears when power is lost:

1. CPU registers and caches.
2. RAM (running processes, network connections, decrypted keys).
3. Network state (ARP tables, routing tables, active connections).
4. Disk artifacts (temporary files, browser sessions).
5. Log files and configuration.
6. Archived backups and cold storage.

Memory capture tools include DumpIt and LiME (Linux), typically run before shutdown.

## Forensic imaging

Imaging creates a bit-for-bit copy of storage so the original can be sealed. The classic tool is `dd`; GUI tools like FTK Imager and `ewfcreate` (EnCase E01 format) add compression and embedded metadata.

```bash
# Linux: image a device to a file, hashing as we go
sudo dd if=/dev/sdb of=/home/evidence/usb-image.dd bs=4M status=progress conv=noerror,sync

# Verify integrity with SHA-256
sha256sum /dev/sdb        # hash the source (before imaging, or a second pass)
sha256sum /home/evidence/usb-image.dd
```

Rules of thumb:

- Use `conv=noerror,sync` so unreadable sectors do not abort the copy.
- Write the image to different physical media than the source.
- Record both hashes in the chain-of-custody log; they must match (for `dd` images, the source hash is computed separately since padding changes length on `noerror` runs).
- Prefer E01/AFF formats for casework: they embed hash values and compress.

## Timeline analysis

Timeline analysis reconstructs events across artifacts by normalizing timestamps into a single chronological view. Watch for UTC vs. local time and for entries that were deliberately altered (a file "created" before the disk was formatted is a red flag).

Sources of timestamps: file MAC times (modify, access, change), registry key LastWrite times, log entries, browser history, email headers, USB device connection records.

Tools: plaso/log2timeline (creates a super-timeline), Timesketch for collaborative review, or simple sorting in a spreadsheet for small cases.

```bash
# Generate a timeline from a disk image
log2timeline.py --storage-file case.plaso --image /evidence/usb-image.dd
psort.py -o csv -w case-timeline.csv case.plaso "date >= '2026-01-01'"
```

## Artifact sources

| Question | Artifact | Where to look |
| --- | --- | --- |
| Which sites were visited? | Browser history | Chrome: `History` (SQLite); Firefox: `places.sqlite` |
| What was typed into search bars? | Search/autocomplete records | Browser history keywords table, Windows Search |
| What ran and when? | Execution artifacts | Windows Registry `UserAssist`, `Amcache`, `BAM` |
| Was a USB drive plugged in? | USB device traces | Windows Registry `SYSTEM\CurrentControlSet\Enum\USBSTOR` |
| Did an account authenticate? | Authentication logs | Windows Event IDs 4624/4625; Linux `/var/log/auth.log` |
| What changed on the system? | File system journal | NTFS `$MFT`, ext4 journal, `$UsnJrnl:$J` |
| Was data exfiltrated? | Network logs | Proxy logs, firewall logs, DNS query history |

## Memory vs. disk forensics

| Dimension | Memory (RAM) forensics | Disk forensics |
| --- | --- | --- |
| Volatility | Lost at power-off | Persists |
| Speed | Fast-moving snapshot | Historical record |
| Reveals | Live connections, keys, injected code, passwords in cleartext | Files, registry, logs, execution history |
| Typical tools | Volatility 3, Rekall-style plugins | Autopsy, The Sleuth Kit, FTK Imager |
| Main risk | Incomplete capture | Timestamp tampering |

A balanced investigation uses both: memory shows what the process was doing, disk shows what it left behind.

## Anti-forensics awareness

Attackers may try to cover tracks; knowing the techniques helps you detect them:

- **Timestomping**: modifying MAC times (e.g., Windows `timestomp`, Linux `touch -d`). Detect via MFT entry mismatch between `$STANDARD_INFORMATION` and `$FILE_NAME` attributes, and by comparing with backup or journal data.
- **Secure delete / wiping**: tools like `shred` overwrite file contents. On SSDs with wear leveling, the original blocks may still exist in unmapped areas — but "the SSD always keeps everything" is a myth too: encrypted SSDs discard data when the ATA erase or crypto-erase happens. Treat overwriting as a reason to look for journal, snapshot, and backup copies rather than assuming data is unrecoverable.
- **Log clearing**: wiping `/var/log` or clearing the Windows Security log (Event ID 1102 records the clearing itself).
- **Fileless techniques**: activity that lives only in memory, raising the value of RAM capture.
- **Slack space and alternate data streams**: hiding data where naive copies will not look.

## Worked scenario: recover a deleted file

Goal: recover a contract PDF a user deleted from a Windows workstation.

1. **Authorize and document.** Confirm scope in writing, open a chain-of-custody record, note the workstation's serial number and current time.
2. **Capture volatile data.** Run a RAM capture (e.g., DumpIt) while the machine is on; record open handles and network connections.
3. **Image the disk.** With the machine off, connect the drive read-only (hardware write blocker preferred) and image it with FTK Imager or `dd`, computing SHA-256 of the image.
4. **Work on the copy.** Mount or load the image in Autopsy/FTK; never analyze the original.
5. **Locate the file.** Check the Recycle Bin (`$Recycle.Bin`, metadata in `$I` files, data in `$R` files). If not there, parse the `$MFT` for a non-allocated entry: the filename remains even after deletion.
6. **Recover the data.** If the MFT still points to valid clusters, export the file and verify it opens. If clusters are partially overwritten, attempt carvers (Autopsy, TestDisk, PhotoRec) on unallocated space.
7. **Verify and report.** Hash the recovered file, note its MAC times and the cluster state, and document every step so another examiner could repeat the result.

Recovery often succeeds because deletion usually only marks space as free; the content survives until the operating system reuses the cluster.

## See also

- [Incident Response Basics](incident-response.md)
- [Building a Home Lab](hands-on-lab.md)
- [Operating System Security](../fundamentals/operating-system-security.md)
- [Threat Landscape](../fundamentals/threat-landscape.md)
