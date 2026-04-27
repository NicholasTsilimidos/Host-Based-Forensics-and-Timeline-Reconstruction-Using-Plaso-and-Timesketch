# M57 Patents Scenario — Extended Background

This document provides additional context beyond what's in the main [README](../README.md). For the project overview and our methodology, refer there first.

## About Digital Corpora

[Digital Corpora](https://digitalcorpora.org/) is a public collection of forensic datasets curated by Simson Garfinkel for digital forensics research and education. The M57 Patents Scenario is one of its most well-known scenarios.

## Why M57 Was Designed This Way

The scenario was built to provide **realistic, multi-source evidence over time**. Key design choices that make it valuable for timeline analysis:

- **Daily disk imaging** — captures temporal change, not just a single moment
- **Multiple users on a shared network** — enables cross-source correlation
- **Both workstation and removable media** — mirrors real corporate environments
- **Real noise** — captured from actual user activity, not synthetic data, which means analysts have to filter signal from genuine background activity

## The Two Main Scenarios

The M57 dataset supports two distinct investigation paths:

| Scenario | Focus |
|---|---|
| **Part 1 — Inappropriate Material** | Discovery of inappropriate digital content on a company computer |
| **Part 2 — Exfiltration of Corporate IP** | Theft of proprietary research by a malicious insider (the scenario we used) |

The scenario document for Part 2 is available in this folder as [`exfiltration-scenario.pdf`](./exfiltration-scenario.pdf).

## Evidence in the Full Dataset (Beyond What We Used)

We worked with disk images only. The full dataset also includes:

- **Network packet captures (PCAP)** — would help identify the outside contact
- **Email server data** — server-side mailbox storage
- **Daily snapshots** of every workstation across the full 5-week scenario period

We deliberately limited our scope to host-based forensics. A complete real-world investigation would incorporate the network and email server data as well.

## Citation

Garfinkel, S., et al. (2009). "Bringing Science to Digital Forensics with Standardized Forensic Corpora." *Digital Investigation*, Vol. 6.
