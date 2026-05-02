# IOC Evidence Timeline

This document maps each event in the investigation timeline to the specific Plaso parser that produced the evidence. For the high-level findings, see [`investigation-summary.md`](./investigation-summary.md).

| Date | Event | Plaso Parser |
|---|---|---|
| Nov 20, 2009 | 18 PDFs copied to `XFER-11-20-2010` folder on work USB | `filestat` |
| Nov 20, 2009 | Hotmail accessed via Internet Explorer | `msiecf` |
| Nov 23, 2009 | Python 2.6 + email libraries installed | `filestat` |
| Nov 23, 2009 | `E-mail.lnk` shortcut created on Jo's desktop | `filestat` |
| Nov 25, 2009 | Hotmail accessed second time | `msiecf` |
| Dec 09, 2009 | `Inbox.dbx` modified (active email activity) | `usnjrnl` |
| Dec 10, 2009 | 4,794 PDFs copied across 17 folders (`Papers1`–`Papers17`) | `filestat` |
| Dec 11, 2009 | Outlook Express launched 14 times (`MSIMN.EXE` run count 14) | `prefetch` |
| Dec 11, 2009 | Hotmail `newmail[1]` artifact present | `filestat` |

Every event in this timeline is reproducible from the original disk images using the commands in the main [README](../README.md#11-how-to-reproduce). To verify any specific IOC, run Plaso against the relevant image and search the resulting CSV for the parser and timestamp.
