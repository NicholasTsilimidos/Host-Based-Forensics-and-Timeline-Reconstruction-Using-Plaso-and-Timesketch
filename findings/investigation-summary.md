# Investigation Summary — Detailed Findings

This document expands on the high-level findings in the main [README](../README.md#6-investigation-findings) by mapping each conclusion back to the specific Plaso parsers and disk artifacts that produced it.

For the chronological event timeline, see [`ioc-timeline.md`](./ioc-timeline.md).

---

## Suspect Identification — Artifact-Level Detail

Jo Smith was identified through correlation of multiple parser outputs:

| Artifact | Plaso Parser | Path |
|---|---|---|
| User registry hive | `winreg/winreg_default` | `HKEY_CURRENT_USER` mapped to user `Jo` |
| User profile directory | `filestat` | `C:\Documents and Settings\Jo\` |
| Application execution context | `prefetch` | All suspicious app launches occurred under Jo's profile |
| Email shortcut | `filestat` | `Documents and Settings/Jo/Desktop/E-mail.lnk` |

---

## Method of Exfiltration — Supporting Artifacts

### USB Drive (Primary Channel)

Both exfiltration stages produced filesystem artifacts on `jo-work-usb-2009-12-11.E01`:

```
2009-11-20  XFER-11-20-2010/         18 PDFs   (Stage 1)
2009-12-10  Papers1/                 282 PDFs
2009-12-10  Papers2/                 282 PDFs
            ...
2009-12-10  Papers17/                282 PDFs  (Stage 2 — 4,794 total)
```

All file creation timestamps were captured by Plaso's `filestat` parser.

### Personal Webmail (Communication Channel)

| Date | Artifact | Parser |
|---|---|---|
| Nov 20 | Hotmail cache files in IE Content.IE5 directory | `msiecf` |
| Nov 25 | Hotmail cache files (second session) | `msiecf` |
| Dec 11 | `newmail[1]` artifact | `filestat` |

### Email Tooling Installed (Capability Indicator)

```
2009-11-23  /Python26/Lib/email/        (directory created)
2009-11-23  /Python26/Lib/email/mime/   (directory created)
2009-11-23  Documents and Settings/Jo/Desktop/E-mail.lnk
```

Captured by `filestat`. Provides the technical capability to send bulk emails programmatically — though we cannot confirm Python was used for this purpose from disk artifacts alone.

---

## Cover-Up Indicators — Forensic Detail

### Outlook Express Activity Spike

The Plaso `prefetch` parser captured this artifact for December 11, 2009:

```
Prefetch [MSIMN.EXE] was executed - run count 14
Path hints: \PROGRAM FILES\OUTLOOK EXPRESS\MSIMN.EXE
```

A run count of 14 in a single day is extraordinary for an email client. Combined with the timing (police arrival day), this strongly indicates email cleanup activity.

### Inbox.dbx Modification

```
2009-12-09  usnjrnl  Inbox.dbx  USN_REASON_BASIC_INFO_CHANGE
```

The USN Journal recorded modification of the local Outlook Express mailbox two days before police arrived.

> **Limitation:** Plaso parses `.dbx` files at the filesystem level only. Message content inside the mailbox would require a specialized parser like `libpff`.

---

## USB Comparison — Eliminating Other Suspects

| Pattern | Jo's Work USB | Charlie's USB | Terry's USB |
|---|---|---|---|
| Proprietary research PDFs | 4,812 files | None matching | None matching |
| `XFER`-named folders | Yes | None | None |
| Systematic duplication (282×17) | Yes | None | None |
| Pre-police activity spike | Yes (Dec 10) | None | None |
| Mac Spotlight metadata | Present | Absent | Absent |

The absence of these patterns on the comparison USBs eliminates the other employees as suspects.

> **Note:** The Mac metadata on Jo's work USB indicates the drive was also connected to a Mac at some point — possibly Jo's personal device. This may be relevant to the eventual destination of the stolen files.

---

## What Could Not Be Determined

Despite the strength of the host-based evidence, host artifacts alone cannot answer:

- **Identity of the outside contact** — would require parsing `Inbox.dbx` content, network forensics, or legal process against Hotmail
- **Final destination of the files** — physical handoff vs. upload from another device cannot be determined from the work USB
- **Intent** — a timeline shows activity, not motive

These gaps are the reason a complete IR engagement combines host, network, and memory forensics.

---

## Plaso Parsers Used in This Investigation

| Parser | Contribution |
|---|---|
| `filestat` | File creation/modification/access timestamps |
| `prefetch` | Application execution history |
| `winreg/winreg_default` | Windows registry — user identification |
| `usnjrnl` | NTFS USN Journal — change tracking |
| `msiecf` | Internet Explorer cache (Hotmail access) |
| `firefox_cache` | Firefox cache (additional browser activity) |
| `lnk` | Windows shortcut artifacts |
| `winevt` | Windows Event Logs |

Every conclusion in this document is supported by output from one or more of these parsers.
