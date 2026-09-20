---
name: time-frequency-baseline-flagging
description: Use when processing e-MERLIN (or other radio interferometric array) UVFITS
  visibility files whose amplitudes contain radio frequency interference, especially
  at L-band where RFI can reach 20–25% of the band.
when_to_use_negative:
- Input is not radio interferometric visibility data (e.g. already-calibrated images
  or non-radio spectra)
- Data are not in a format ParselTongue/AIPS can load into the AIPS environment (the
  pipeline operates on data in AIPS)
- A non-automated, manual visual inspection workflow is explicitly required rather
  than automated RFI detection
edam_operation: http://edamontology.org/operation_3695
tools:
- name: SERPent
  role: Automated reduction and RFI-mitigation pipeline implementing the SumThreshold-based
    flagging
  repo: https://github.com/daniellefenech/SERPent
- name: Parseltongue
  role: Language used by SERPent to interact directly with data in the AIPS environment,
    including FIT/UVFITS loading and export
- name: AIPS
  role: Astronomical Image Processing Software environment in which the visibility
    data reside and flags are propagated
- name: Python
  role: SERPent is a simple 'out of the box' Python script, free of compilers
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.09.001
    title: 'SERPent: Automated reduction and RFI-mitigation software for e-MERLIN'
  source_content_hash: sha256:b129d42b00ecaf2924f0e38a8b3bd5ae7b5b1ea17be400312916b75ca208c130
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# time-frequency-baseline-flagging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Per-baseline RFI detection on radio interferometric visibility data by reordering amplitudes into time-by-frequency sequences and applying SumThreshold-based classification (Offringa et al. 2010; 2012) as operationalised in the SERPent pipeline for e-MERLIN L-band (1.3–1.8 GHz) and C-band (4–8 GHz) data.

## When to use

Use when processing e-MERLIN (or other radio interferometric array) UVFITS visibility files whose amplitudes contain radio frequency interference, especially at L-band where RFI can reach 20–25% of the band.

## When NOT to use

- Input is not radio interferometric visibility data (e.g. already-calibrated images or non-radio spectra)
- Data are not in a format ParselTongue/AIPS can load into the AIPS environment (the pipeline operates on data in AIPS)
- A non-automated, manual visual inspection workflow is explicitly required rather than automated RFI detection

## Inputs

- UVFITS visibility file from an e-MERLIN commissioning observation (L-band 1.3–1.8 GHz or C-band 4–8 GHz)
- Visibility amplitudes per interferometer baseline, reordered into time-by-frequency sequences

## Outputs

- Flagged UVFITS visibility file, immediately ready for AIPS calibration
- Per-sample RFI flags propagated to AIPS visibility records

## How to apply

Load the UVFITS visibility file into AIPS via ParselTongue, then read visibility amplitudes and reorder them into a time-by-frequency sequence for each interferometer baseline, as required by the SumThreshold formulation of Offringa et al. 2010 (MNRAS, 405, 155-167). Apply SumThreshold per baseline: iteratively test contiguous groups of amplitude samples along both the time and frequency axes against sensitivity-scaled thresholds to classify RFI-affected samples, using the classification and detection algorithms from Offringa et al. 2010 and Offringa et al. 2012 (A&A, 539, A95). Propagate detected flags back to the AIPS visibility records, extending flagging to antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans inherent to the e-MERLIN system. Export the flagged dataset back to UVFITS via ParselTongue so the output is immediately ready for AIPS calibration.

## Related tools

- **SERPent** (Automated reduction and RFI-mitigation pipeline implementing the SumThreshold-based flagging) — https://github.com/daniellefenech/SERPent
- **Parseltongue** (Language used by SERPent to interact directly with data in the AIPS environment, including FIT/UVFITS loading and export)
- **AIPS** (Astronomical Image Processing Software environment in which the visibility data reside and flags are propagated)
- **Python** (SERPent is a simple 'out of the box' Python script, free of compilers)

## Evaluation signals

- Flagged output file exists and is a valid UVFITS file readable back into AIPS
- Flagged data fraction is consistent with reported e-MERLIN RFI levels: <20%–25% for problematic L-band observations and <5% for the quieter C-band
- Antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans are flagged in addition to RFI-affected samples
- Output is immediately ready for AIPS calibration without intermediate manual intervention
- Flagging is more accurate and delicate than visual manual flagging, as claimed for SERPent

## Limitations

- RFI is worst at L-band (1.3–1.8 GHz) where the spectrum is actively used for other purposes; expect higher flagged fractions there than at C-band
- Throughput limited to ~110 GB day−1 on a 'high-end' computer (16 CPUs, 100 GB memory), i.e. ∼6.9 GB CPU−1 day−1
- The script additionally targets e-MERLIN-specific artefacts (antenna zero-amplitude dropouts, Lovell telescope phase calibrator stationary scans), so behaviour on other arrays may omit these steps

## Evidence
