# M57 Patents Scenario — Background

The M57 Patents Scenario is a publicly available forensic dataset published by [Digital Corpora](https://digitalcorpora.org/), curated by Simson Garfinkel for forensic education and research. It simulates a small patent research firm — M57.biz — operating over a five-week period in November and December 2009, with daily disk images captured for each employee workstation along with their associated USB drives.

We chose this scenario because it provides a realistic corporate environment, multi-source evidence, and daily snapshots — making it ideal for timeline reconstruction work. The dataset is also publicly accessible, which means our entire investigation is reproducible by anyone.

The dataset includes two main scenarios. We focused on **Part 2 — Exfiltration of Corporate Intellectual Property**, which centers on the suspicion that an employee has been passing proprietary research to an outside party. The original scenario document is available in [`exfiltration-scenario.pdf`](./exfiltration-scenario.pdf).

The full dataset also includes network packet captures (PCAP) and email server data, which we deliberately did not use — keeping our scope to host-based forensics. A complete real-world investigation would incorporate those additional layers.

**Citation:** Garfinkel, S., et al. (2009). "Bringing Science to Digital Forensics with Standardized Forensic Corpora." *Digital Investigation*, Vol. 6.
