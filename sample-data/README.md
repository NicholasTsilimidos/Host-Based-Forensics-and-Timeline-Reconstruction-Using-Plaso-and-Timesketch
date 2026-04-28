# Sample Data

This directory contains a representative sample from the M57 dataset to allow reviewers to verify our methodology without downloading the full corpus.

We selected Charlie's work USB because it is small enough to host on GitHub and serves as supporting evidence in our investigation. It was used to confirm that Jo's exfiltration pattern was unique.

## Files

| File | Description |
|---|---|
| `charlie-work-usb-2009-12-11.E01` | Source disk image (EnCase format) |
| `charlie-work-usb.plaso` | Plaso storage file produced from the image |
| `charlie-work-usb-timeline.csv` | CSV timeline exported via psort |

## Reproducing These Files

To regenerate the `.plaso` and `.csv` from the source image, see the main [README](../README.md#11-how-to-reproduce). The same commands apply, just point them at this image.

## Full Dataset

The complete M57 Patents Scenario is available at [Digital Corpora](https://digitalcorpora.org/corpora/scenarios/m57-patents-scenario/).

> **Note:** The full dataset is several hundred gigabytes. Plan storage and bandwidth accordingly.
