---
name: rfi-sumthreshold-flagging
description: Use when processing e-MERLIN (or any radio interferometric array) UVFITS
  visibility data where RFI contamination — typically <20%-25% at L-band (1.3-1.8
  GHz) or <5% at C-band (4-8 GHz) — must be flagged before calibration.
when_to_use_negative:
- The input is not radio interferometric visibility data (e.g. already-imaged FITS
  products or non-visibility tables)
- Data cannot be loaded into the AIPS environment — SERPent is written in ParselTongue/Python
  specifically to interact with data in AIPS
- A calibrated, RFI-free dataset is already available and no further flagging is required
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SERPent
  role: Automated reduction and RFI-mitigation pipeline implementing the SumThreshold
    flagging methodology
  repo: https://github.com/daniellefenech/SERPent
- name: Parseltongue
  role: Language in which SERPent is written, enabling interaction with AIPS and FIT-loading/export
    of UVFITS data
- name: AIPS
  role: Astronomical Image Processing Software environment in which the visibility
    data are held, flagged, and made ready for calibration
- name: Python
  role: Scripting language for SERPent, a simple 'out of the box' script free of compilers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.09.001
    title: 'SERPent: Automated reduction and RFI-mitigation software for e-MERLIN'
  source_content_hash: sha256:fe1807636e8129b3e053b5d9efe449bb8f994e027b7d787968954688d0ef4cc6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/daniellefenech/SERPent
  repo_url: https://github.com/daniellefenech/SERPent
  tools:
  - AIPS
  - Parseltongue
  - Python
  - SERPent
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# rfi-sumthreshold-flagging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated RFI-mitigation skill for radio interferometric visibility data that applies the SumThreshold methodology (Offringa et al. 2010a), as operationalised by SERPent in ParselTongue/Python within the AIPS environment for e-MERLIN L-band and C-band datasets.

## When to use

Use when processing e-MERLIN (or any radio interferometric array) UVFITS visibility data where RFI contamination — typically <20%-25% at L-band (1.3-1.8 GHz) or <5% at C-band (4-8 GHz) — must be flagged before calibration.

## When NOT to use

- The input is not radio interferometric visibility data (e.g. already-imaged FITS products or non-visibility tables)
- Data cannot be loaded into the AIPS environment — SERPent is written in ParselTongue/Python specifically to interact with data in AIPS
- A calibrated, RFI-free dataset is already available and no further flagging is required

## Inputs

- UVFITS visibility file from an e-MERLIN commissioning observation (L-band 1.3-1.8 GHz or C-band 4-8 GHz), loadable into AIPS via ParselTongue

## Outputs

- Flagged UVFITS visibility dataset ready for AIPS calibration
- RFI flags covering RFI-affected visibilities, antenna zero-amplitude dropouts, and Lovell telescope phase calibrator stationary scans

## How to apply

Load the e-MERLIN commissioning UVFITS visibility file into AIPS via ParselTongue, then reorder visibility amplitudes into a time-by-frequency sequence per interferometer baseline as required by the SumThreshold formulation of Offringa et al. 2010 (MNRAS, 405, 155-167). Apply SumThreshold per baseline, iteratively testing contiguous groups of amplitude samples along time and frequency axes against sensitivity-scaled thresholds to classify RFI-affected samples, drawing on the classification and detection algorithms from Offringa et al. 2010 and Offringa et al. 2012 (A&A, 539, A95). Propagate detected flags to the AIPS visibility records, and additionally flag antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans inherent to the e-MERLIN system. Export the flagged dataset back to UVFITS via ParselTongue so the output is immediately ready for AIPS calibration; the paper reports this automated detection is more accurate and delicate than visual manual flagging.

## Related tools

- **SERPent** (Automated reduction and RFI-mitigation pipeline implementing the SumThreshold flagging methodology) — https://github.com/daniellefenech/SERPent
- **Parseltongue** (Language in which SERPent is written, enabling interaction with AIPS and FIT-loading/export of UVFITS data)
- **AIPS** (Astronomical Image Processing Software environment in which the visibility data are held, flagged, and made ready for calibration)
- **Python** (Scripting language for SERPent, a simple 'out of the box' script free of compilers)

## Evaluation signals

- Flagged output file exists and is a valid UVFITS file
- Flagged data fraction is consistent with the paper's reported RFI levels: <20%-25% for problematic L-band (1.3-1.8 GHz) observations and <5% for the quieter C-band (4-8 GHz)
- Flagged dataset loads into AIPS and is immediately ready for AIPS calibration
- Antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans present in the input are flagged in the output
- Automated flagging result is more accurate and delicate than a visual manual flagging comparison on the same dataset

## Limitations

- Flagging throughput is 110 GB day^-1 on a 'high-end' computer (16 CPUs, 100 GB memory), i.e. ~6.9 GB CPU^-1 day^-1
- Antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans are e-MERLIN-system-specific artefacts; other arrays may need different extra flagging steps
- Reported RFI fractions (<20%-25% L-band, <5% C-band) characterise e-MERLIN commissioning datasets; other arrays/bands may differ

## Evidence
