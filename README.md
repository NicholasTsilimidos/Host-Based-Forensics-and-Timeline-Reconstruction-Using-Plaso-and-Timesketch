# Host-Based Forensics & Timeline Reconstruction Using Plaso + Timesketch

A forensic investigation of the Digital Corpora 2009 M57 Patents scenario, demonstrating how open-source timeline reconstruction tools support real-world incident response.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Why This Project Matters](#2-why-this-project-matters)
3. [Dataset & Scenario](#3-dataset--scenario)
4. [Tools Used](#4-tools-used)
5. [Methodology](#5-methodology)
6. [Investigation Findings](#6-investigation-findings)
7. [Court Defensibility](#7-court-defensibility)
8. [Challenges & Lessons Learned](#8-challenges--lessons-learned)
9. [Conclusion](#9-conclusion)
10. [Repository Structure](#10-repository-structure)
11. [How to Reproduce](#11-how-to-reproduce)
12. [References](#12-references)

---

## 1. Project Overview

This project focuses on host-based forensic analysis and timeline reconstruction using the Digital Corpora 2009 M57 Patents Scenario dataset. The objective was to analyze a series of forensic disk images and reconstruct system activity by extracting timestamped artifacts from the host.

Using **Plaso (log2timeline)**, we processed the disk images to generate a detailed timeline of events including file system metadata, Windows event logs, registry activity, prefetch data, and browser artifacts. These artifacts were then used to identify patterns of behavior and indicators of compromise (IOCs).

The generated timelines were further visualized and analyzed using **Timesketch**, allowing us to correlate events across multiple sources and reconstruct exactly how activity occurred on the system. This approach supports the incident response process by helping determine the scope of an incident and providing evidence-based conclusions.

By the end of the project, we were able to identify a suspect, confirm the method of exfiltration, document the full scale of the data theft, and produce a court-traceable timeline of activity, all from disk artifacts alone.

---

## 2. Why This Project Matters

In real-world incident response, understanding **what happened, when it happened, and in what order** is fundamental. A forensic timeline is often the difference between a vague suspicion and a defensible conclusion.

This project demonstrates several principles that apply directly to professional IR practice:

- **Open-source tools are production-ready:** Plaso and Timesketch are used by forensic labs and law enforcement worldwide, not just in academic settings.
- **Timeline forensics scales:** A single disk image can contain millions of timestamped events. Manual analysis is not viable, automated extraction and visualization are essential.
- **Cross-source correlation strengthens evidence:** When the same event appears on multiple independent sources at matching timestamps, the evidentiary value increases substantially.
- **Host forensics is one layer of a complete investigation:** The full picture requires network forensics and memory analysis alongside host-based evidence.

Understanding both the strengths and the limitations of these tools is critical for any analyst preparing forensic findings that may be used in legal proceedings.

---

## 3. Dataset & Scenario

### About the Dataset

We used the **2009 M57 Patents Scenario**, a publicly available forensic training corpus published by [Digital Corpora](https://digitalcorpora.org/corpora/scenarios/m57-patents-scenario/). The dataset simulates a small patent research firm operating over a five-week period, with daily disk images captured for each employee workstation along with their associated USB drives.

We chose this dataset for several reasons:

- **Realistic environment:** The scenario simulates a functioning corporate setting rather than a synthetic CTF challenge.
- **Multi-source evidence:** Workstations and USB drives provide opportunities for cross-source correlation.
- **Temporal depth:** Daily snapshots over multiple weeks allow for meaningful timeline analysis.
- **Publicly available:** Anyone can download the dataset and reproduce our work.
- **Well-studied:** The scenario has been used in academic research, allowing us to validate our findings against known outcomes.

> **Note on dataset size:** The full M57 dataset is several hundred gigabytes. We have not included the full disk images in this repository. Instead, we have provided a **sample USB image** (Charlie's work USB) along with its corresponding `.plaso` file and CSV timeline in the [`sample-data/`](./sample-data/) directory. The complete dataset is available at the link above.

### The Exfiltration Scenario

The full scenario document is available in [`docs/exfiltration-scenario.pdf`](./docs/M57-Patents-Exfiltration-Scenario.pdf).

> **M57.biz** is a patent research firm with four employees: Pat (CEO), Terry (IT), and two patent researchers, Jo and Charlie. Over the course of November and December 2009, proprietary client research was being passed to an outside party. On December 11, 2009, police seized all workstations and USB drives. Our task was to identify who was responsible, how they did it, and what was taken, using only the host-based disk artifacts.

### Evidence Processed

| Source | Description |
|---|---|
| Jo Smith - Workstation | Windows XP workstation, primary suspect machine |
| Jo Smith - Personal USB | Seized by warrant, contained personal files |
| Jo Smith - Work USB | Company-issued USB drive |
| Pat McGoo - Workstation | CEO workstation |
| Charlie Brown - Workstation | Co-researcher workstation |
| Charlie Brown - Work USB | Company-issued USB drive |
| Terry Johnson - Workstation | IT staff workstation |
| Terry Johnson - Work USB | Company-issued USB drive |

Total: **8 disk images** processed.

---

## 4. Tools Used

### Plaso (log2timeline)

Plaso is an open-source forensic framework, originally developed by Kristinn Gudjonsson and now maintained by Google. It extracts and normalizes timestamped events from forensic disk images into a single unified storage format (`.plaso`).

**Key capabilities:**
- 800+ parsers covering Windows Event Logs, registry hives, file system metadata (`$MFT`), prefetch, browser history, LNK files, USB history, and more
- Two main commands: `log2timeline.py` (extraction) and `psort.py` (export and filtering)
- Native integration with Timesketch via the `timesketch_importer` utility
- Cross-platform support (Linux, macOS via Docker, Windows)

**Limitations we encountered:**
- Cannot read the body content of email storage formats (`.dbx`, `.pst`), only metadata
- Cannot recover deleted file contents, only metadata about deleted files
- Processing is single-threaded by default; a single large disk image can take 45–90 minutes
- Susceptible to timestomping (deliberate timestamp manipulation by an attacker)

### Timesketch

Timesketch is an open-source web-based platform for collaborative forensic timeline analysis, also developed by Google. It ingests `.plaso` files and provides a searchable, taggable, annotatable interface for investigation.

**Key capabilities:**
- Search across millions of events using keywords, date ranges, and parser filters
- Tag and annotate suspicious events for organized investigation
- Write linked **Stories** that connect prose narrative directly to timestamped evidence
- Run **Sigma rules** for automated threat detection across timelines
- Multi-analyst collaboration on a single sketch
- Backend powered by OpenSearch for scalability to billions of events

**Limitations we encountered:**
- Multi-container deployment (OpenSearch, PostgreSQL, Redis, Nginx) is non-trivial to configure
- No built-in support for network forensics (PCAP) or memory analysis
- Quality of analysis depends on quality of input data
- Steep initial learning curve for new investigators

---

## 5. Methodology

Our investigation followed six phases. The full reproduction commands are in [Section 11](#11-how-to-reproduce).

### Phase 1 - Environment Setup

Plaso was run via Docker. We tested both macOS (Apple Silicon, with `linux/amd64` emulation) and a Kali Linux VM. The Linux VM was significantly more stable and faster.

Timesketch was deployed via its official Docker Compose configuration on a Windows host after initial setup attempts on macOS proved problematic.

### Phase 2 - Artifact Extraction with Plaso

We ran `log2timeline.py` against each of the 8 disk images. Several flags were essential:

- `--partitions all` - required for the multi-partition M57 images
- `--vfs_back_end tsk` - uses The Sleuth Kit backend for stability under Docker emulation
- `--single_process` - prevents worker crashes on emulated environments

Each extraction produced a `.plaso` storage file containing every timestamped artifact found on the image.

### Phase 3 - Timeline Export with psort

Each `.plaso` file was exported to a CSV timeline using `psort.py` with the `dynamic` output format. This produced human-readable timelines that could be further filtered and analyzed.

### Phase 4 - Filtering and Analysis

The raw CSV timelines were too large to analyze manually (millions of events per image). We wrote targeted Python scripts to filter the data by:
- Date ranges (focusing on November–December 2009)
- Parser type (e.g., `prefetch`, `usnjrnl`, `msiecf`)
- Keywords related to the investigation (e.g., `XFER`, `Papers`, `Outlook`, `Hotmail`, `MSIMN`)

### Phase 5 - Timesketch Visualization

The `.plaso` files were imported into a single Timesketch sketch as separate named timelines. This allowed us to:
- Search across all sources simultaneously
- Tag suspicious events with custom labels (`exfiltration`, `cover-up`, `webmail-access`)
- Apply date-range filters to isolate critical windows (e.g., December 10–11)
- Compare suspect and non-suspect USBs side-by-side
- Build a Story narrative linking the evidence chronologically

### Phase 6 - Findings Documentation

All findings were documented with traceability to their source artifacts and timestamps. See [Section 6](#6-investigation-findings) for the summary and the [`findings/`](./findings/) directory for the full breakdown.

---

## 6. Investigation Findings

> Full details with screenshots are available in [`findings/`](./findings/).

### Suspect Identified

**Jo Smith** - Patent Researcher at M57.biz (`jo@m57.biz`)

Confirmation came from multiple Plaso parsers across the suspect's workstation timeline: registry artifacts (`HKEY_CURRENT_USER`), the user profile path (`C:\Documents and Settings\Jo\`), and prefetch artifacts.

### Method of Exfiltration

**Two-stage USB exfiltration**, supported by webmail communication:

| Date | Activity |
|---|---|
| Nov 20, 2009 | 18 patent PDFs copied to a folder named `XFER-11-20-2010` on Jo's work USB. Hotmail accessed via web browser. |
| Nov 23, 2009 | Python 2.6 installed with email libraries; email shortcut created on the desktop. |
| Nov 25, 2009 | Hotmail accessed a second time. |
| Dec 09, 2009 | `Inbox.dbx` modified (Outlook Express). |
| Dec 10, 2009 | **4,794 PDFs copied** across 17 folders (`Papers1` through `Papers17`), bulk exfiltration the day before police arrived. |
| Dec 11, 2009 | Outlook Express launched 14 times and consistent with deleting sent email evidence. |

### Scale of Theft

A total of **4,812 proprietary patent research PDFs** were copied to Jo's work USB. The file naming pattern (`[ID].[Topic].[Author].pdf`) matched M57.biz's client deliverables. The same 282-paper set was duplicated across 17 folders on December 10, indicating systematic mass copying.

### Supporting Evidence

Cross-comparison with Charlie's and Terry's work USBs revealed **no matching folder structures, no bulk PDF duplication, and no transfer-named folders**. This isolates Jo as the sole exfiltrator and eliminates the other employees as suspects.

### Cover-up Indicators

- 14 launches of Outlook Express on the day of police arrival (Dec 11)
- Use of personal Hotmail webmail to bypass company email logging
- `Inbox.dbx` modification two days before seizure

---

## 7. Court Defensibility

### What We Established

Plaso and Timesketch operate within the **Detection and Analysis** phase of the incident response lifecycle. From disk artifacts alone, we were able to establish *who* the suspect was, *what* was taken, *how* it was moved, and *when* the activity occurred. Every finding ties back to a specific artifact and timestamp, no conclusions were assumed or extrapolated.

### What Strengthens the Case

- **Read-only processing.** All disk images were processed without modification, preserving the original evidence.
- **Reproducible methodology.** Every command and flag is documented in this repository. Any analyst can run the same commands and produce identical output.
- **Cross-source corroboration.** The same exfiltration events appear on both Jo's workstation and Jo's work USB at matching timestamps, evidence from two independent sources.

### What's Still Missing

- We can confirm that files were copied to a USB drive, we cannot prove where they went after that.
- The outside contact could not be identified from the disk image alone.
- A timeline demonstrates *activity*, not *intent*. The intent argument must be made in court by a human.

### Bottom Line

Host forensics tells you what happened on the machine. **Network forensics** tells you where the data went and to whom. **Memory forensics** reveals what was running at the time that never reached disk. A complete investigation requires all three layers. What this project demonstrates is a strong foundation built on the host layer.

---

## 8. Challenges & Lessons Learned

### Setup & Environment

- **Plaso on macOS is functional but limited.** Apple Silicon requires `linux/amd64` emulation under Docker, which introduces stability and performance issues. We strongly recommend running Plaso on a native Linux environment such as a Kali VM.
- **Timesketch installation on macOS was problematic.** The multi-container stack (OpenSearch, PostgreSQL, Redis, Nginx) failed to deploy cleanly. Moving to Windows resolved the setup issues immediately.

### Working With the Data

- **CSV timelines are massive.** A single disk image can produce millions of events. Manual review of raw CSVs is not viable. Always use either Python filtering or Timesketch's search interface.
- **Timestamps can be manipulated.** *Timestomping* is the deliberate alteration of file timestamps by an attacker. Always treat timestamps as evidence subject to scrutiny rather than ground truth.

### Processing & Performance

- **Plaso extraction is slow.** A single large disk image can take 45–90 minutes. We ran extractions in parallel across multiple terminals to reduce total processing time.
- **Monitor system resources.** Docker provides real-time CPU and memory monitoring. Use it to avoid overloading the host system when running multiple extractions.

### Tool Limitations

- **Plaso cannot parse email content.** It captures `.dbx` and `.pst` files as metadata only, it does not extract message bodies. This was a real gap in our investigation, as the outside contact's identity likely lived in those messages.
- **Timesketch alone is incomplete.** A full investigation requires network forensics (PCAP analysis) and memory forensics (RAM dump analysis) alongside host-based timeline reconstruction.

---

## 9. Conclusion

When we started this project, we had a disk image, two tools, and a scenario. By the end of it, we had a suspect, a complete timeline, and a story that the evidence told on its own. That is what host-based forensics is, not building a case from scratch, but reading one that already exists in the artifacts.

Every file copy, every Outlook Express launch, every Hotmail session left a trace. Plaso pulled those traces out of 8 different sources. Timesketch let us connect them together and make sense of them as a unified investigation.

The tools work. They are not perfect, we ran into limitations, and there are questions this investigation could not answer. But what we have is solid, documented, and traceable. In a real incident, that is the foundation everything else gets built on.

**Host forensics is the starting point. Not the finish line.**

---

## 10. Repository Structure

```
m57-host-forensics/
│
├─ README.md                          ← this file
│
├─ docs/
│   ├─ exfiltration-scenario.pdf      ← original scenario document
│   └─ m57-background.md              ← extended dataset and company background
│
├─ sample-data/
│   ├─ README.md                      ← explains sample files and dataset access
│   ├─ charlie-work-usb-2009-12-11.E01
│   ├─ charlie-work-usb.plaso
│   └─ charlie-work-usb-timeline.csv
│
├─ findings/
│   ├─ investigation-summary.md       ← detailed findings: who, what, how, cover-up
│   ├─ ioc-timeline.md                ← indicators of compromise timeline
│   ├─ timesketch-analysis.md         ← timesketch screenshots with analysis
│   └─ screenshots/                   ← timesketch screenshots referenced above
│
└─ presentation.pptx                  ← final presentation file
```

---

## 11. How to Reproduce

### Prerequisites

- **Docker Desktop** (for Plaso) - [docker.com](https://www.docker.com/)
- **Python 3.8+** (for filtering scripts)
- **Timesketch** instance - see [official Timesketch documentation](https://timesketch.org/guides/admin/install/) for installation
- The M57 dataset - download from [Digital Corpora](https://digitalcorpora.org/corpora/scenarios/m57-patents-scenario/)

### Recommended Environment

Based on our experience, we recommend:
- **Plaso:** Native Linux (Ubuntu, Kali) - avoid macOS if possible
- **Timesketch:** Linux or Windows host - avoid macOS

### Step 1 - Set Up Project Directory

Create a project directory with the following structure:

```bash
mkdir -p ~/m57-project/{datasets,plaso-output,csv-files}
mkdir -p ~/m57-project/datasets/{drives,usb}
```

> **Note:** Replace `~/m57-project` throughout this guide with your actual project path.

Place the M57 disk images in:
- Workstation images → `~/m57-project/datasets/drives/`
- USB images → `~/m57-project/datasets/usb/`

### Step 2 - Pull the Plaso Docker Image

```bash
docker pull log2timeline/plaso
```

Verify the installation:

```bash
docker run --rm log2timeline/plaso log2timeline.py --version
```

### Step 3 - Extract Artifacts with Plaso

Run the following command **for each disk image**. Adjust the input file path and output filename as needed.

```bash
docker run --rm --platform linux/amd64 \
  -v ~/m57-project:/data \
  log2timeline/plaso \
  log2timeline.py --storage_file /data/plaso-output/charlie-work-usb.plaso \
  --partitions all --vfs_back_end tsk --single_process \
  /data/datasets/usb/charlie-work-usb-2009-12-11.E01
```

**What each flag does:**

| Flag | Purpose |
|---|---|
| `--platform linux/amd64` | Forces Intel emulation under Docker (required on Apple Silicon) |
| `-v ~/m57-project:/data` | Mounts your project directory into the container, **make sure to adjust the host path to match yours** |
| `--storage_file` | Output `.plaso` file path |
| `--partitions all` | Processes all partitions (required for multi-partition images) |
| `--vfs_back_end tsk` | Uses The Sleuth Kit VFS backend (more stable under emulation) |
| `--single_process` | Disables multi-threading (avoids worker crashes on emulated environments) |

> **Tip:** To process multiple images in parallel, open separate terminal windows and run a different image in each. Monitor your system resources via Docker Desktop to avoid overload.

### Step 4 - Export to CSV with psort

```bash
docker run --rm --platform linux/amd64 \
  -v ~/m57-project:/data \
  log2timeline/plaso \
  psort.py -o dynamic \
  -w /data/csv-files/charlie-work-usb-timeline.csv \
  /data/plaso-output/charlie-work-usb.plaso
```
> **Optional Step:** The raw CSVs are too large to analyze directly. Use Python to create email filtering script to reduce the load.

### Step 5 - Import to Timesketch

Once your Timesketch instance is running, import each `.plaso` file as a separate timeline:

```bash
timesketch_importer \
  --host http://localhost:5000 \
  --username admin \
  --timeline_name "Charlie Work USB" \
  ~/m57-project/plaso-output/charlie-work-usb.plaso
```

Repeat for each of the 8 `.plaso` files, naming each timeline appropriately (e.g., "Jo Work USB", "Charlie Machine").

### Step 6 - Investigate in Timesketch

Recommended search queries to reproduce our key findings:

| Query | What It Reveals |
|---|---|
| `XFER` | First-stage exfiltration folder (Nov 20) |
| `Papers` | Bulk exfiltration folders (Dec 10) |
| `MSIMN` | Outlook Express prefetch artifacts |
| `Hotmail` | Webmail access events |
| `Inbox.dbx` | Email database modifications |

Apply date filters (`2009-12-10` to `2009-12-11`) to isolate the critical 48-hour window.

---

## 12. References

### References

- **Digital Corpora: M57 Patents Scenario.** [https://digitalcorpora.org/corpora/scenarios/m57-patents-scenario/](https://digitalcorpora.org/corpora/scenarios/m57-patents-scenario/)
- **Plaso Documentation.** [https://plaso.readthedocs.io/](https://plaso.readthedocs.io/)
- **Timesketch Documentation.** [https://timesketch.org/](https://timesketch.org/)
- **The Sleuth Kit (TSK).** [https://www.sleuthkit.org/](https://www.sleuthkit.org/)
- **NIST SP 800-86 - Guide to Integrating Forensic Techniques into Incident Response.** [https://csrc.nist.gov/publications/detail/sp/800-86/final](https://csrc.nist.gov/publications/detail/sp/800-86/final)
