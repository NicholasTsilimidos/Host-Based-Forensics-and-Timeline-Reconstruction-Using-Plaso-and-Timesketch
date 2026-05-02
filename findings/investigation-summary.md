# Investigation Summary

Our analysis identified **Jo Smith**, a patent researcher at M57.biz, as the sole suspect responsible for exfiltrating proprietary client research. The conclusion was reached entirely from disk artifacts using Plaso parsers across Jo's workstation and work USB.

The exfiltration occurred in two stages. On **November 20, 2009**, Jo copied 18 patent PDFs to a folder explicitly named `XFER-11-20-2010` on the work USB. On **December 10, 2009**, the day before police arrived, Jo copied a total of **4,794 PDFs** organized into 17 identical folders (`Papers1` through `Papers17`), each containing the same 282 papers. The systematic duplication suggests scripted or automated copying. In total, **4,812 proprietary research documents** were stolen.

Jo also used personal Hotmail webmail (accessed November 20 and 25) to communicate outside company email monitoring. Python 2.6 with email libraries was installed on November 23, providing the technical capability to send bulk emails programmatically. On December 9, the local Outlook Express mailbox (`Inbox.dbx`) was modified, suggesting active email exchange just before the seizure.

On **December 11**, the day police arrived, Outlook Express was launched **14 times** (confirmed via Plaso's prefetch parser). This is consistent with deletion of sent emails to remove evidence of the outside contact.

Comparison with Charlie's and Terry's work USBs revealed no matching patterns, no XFER folders, no bulk PDF duplication, no pre-police activity spike. This isolates Jo as the sole exfiltrator.

What we could **not** determine from disk artifacts alone: the identity of the outside contact, the final destination of the files, and Jo's intent. Identifying the contact would require parsing `Inbox.dbx` content, network forensics, or legal process against Hotmail. These gaps are why a complete IR engagement combines host, network, and memory forensics.

For the chronological event-by-event timeline with parser mappings, see [`ioc-timeline.md`](./ioc-timeline.md).
