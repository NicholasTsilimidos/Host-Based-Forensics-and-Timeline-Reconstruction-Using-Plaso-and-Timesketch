# IOC Evidence Timeline

This document provides the artifact-level evidence for each event in our investigation timeline. For high-level findings, see [`investigation-summary.md`](./investigation-summary.md). For the full case overview, see the main [README](../README.md).

---

## November 20, 2009 — First Transfer + Webmail Access

**Activity:** 18 patent PDFs copied to `XFER-11-20-2010` folder on Jo's work USB. Hotmail accessed via Internet Explorer.

| Parser | Evidence |
|---|---|
| `filestat` | 18 PDF creation timestamps in `XFER-11-20-2010/` |
| `msiecf` | `Hotmail[1]`, `WL_Hotmail_24x22_blue[1].gif` cached |
| `prefetch` | `IMAPI.EXE` executed (CD burning service — possibly relevant) |

The folder name `XFER` (transfer) is itself an indicator of intent.

---

## November 23, 2009 — Email Tooling Installed

**Activity:** Python 2.6 with email libraries installed; email shortcut created on desktop.

| Parser | Evidence |
|---|---|
| `filestat` | `/Python26/Lib/email/` directory created |
| `filestat` | `Documents and Settings/Jo/Desktop/E-mail.lnk` created |

Capability indicator — three days after the first XFER transfer.

---

## November 25, 2009 — Second Webmail Access

**Activity:** Hotmail accessed again via web browser.

| Parser | Evidence |
|---|---|
| `msiecf` | `mbox[1].js`, `WindowsLive_Hotmail_logo_128` cached |

Establishes a pattern of webmail use rather than a one-off event.

---

## December 9, 2009 — Email Activity Pre-Police

**Activity:** Outlook Express mailbox file (`Inbox.dbx`) modified.

| Parser | Evidence |
|---|---|
| `usnjrnl` | `Inbox.dbx` — `USN_REASON_BASIC_INFO_CHANGE` |

Active email exchange two days before seizure.

---

## December 10, 2009 — Bulk Exfiltration

**Activity:** 4,794 patent PDFs copied across 17 folders (`Papers1` through `Papers17`) on Jo's work USB.

| Parser | Evidence |
|---|---|
| `filestat` | 4,794 file creation timestamps across 17 folders |
| `filestat` | 17 directory creation timestamps for `Papers1`–`Papers17` |

The day before police arrived. Same 282 papers duplicated 17 times indicates scripted/automated copying.

---

## December 11, 2009 — Police Arrive + Cover-Up

**Activity:** Outlook Express launched 14 times; Hotmail accessed.

| Parser | Evidence |
|---|---|
| `prefetch` | `MSIMN.EXE` run count 14 |
| `msiecf` | `frntpage.htm` — 13 hits |
| `filestat` | `newmail[1]` artifact in IE cache |

Run count of 14 in a single day is extraordinary and consistent with deletion activity.

---

## Verifying These IOCs

Each IOC can be independently verified by running Plaso against the original disk image and grepping the resulting CSV. Example:

```bash
grep "2009-12-10" jo-work-usb-timeline.csv | grep "Papers" | wc -l
```

This should return ~4,794 — confirming the bulk copy event count.

For full reproduction instructions, see Section 11 of the main [README](../README.md#11-how-to-reproduce).

---

## Note on Timestamp Reliability

All timestamps come from disk artifacts as parsed by Plaso. In a court setting, the possibility of timestomping (deliberate timestamp manipulation by an attacker) should be acknowledged. Cross-correlation across multiple independent parsers — as we have here — strengthens confidence in the timeline.
