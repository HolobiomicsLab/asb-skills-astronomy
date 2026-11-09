---
name: uvfits-data-manipulation
description: Use when the input is a radio-interferometric visibility dataset in UVFITS
  format that must be loaded into AIPS, restructured per baseline for RFI flagging
  (e.g. SumThreshold), and exported back ready for calibration.
when_to_use_negative:
- The input is not radio interferometric visibility data (SERPent's scope is 'any
  radio interferometric array data', not single-dish or non-radio data).
- The data are already flagged/calibrated and the goal is further imaging or analysis
  rather than RFI-mitigation-style flagging and re-export of UVFITS files.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: SERPent
  role: Automated reduction and RFI-mitigation pipeline that performs the UVFITS load,
    per-baseline reordering, flagging and re-export.
  repo: daniellefenech/SERPent
- name: Parseltongue
  role: Language used by SERPent to load UVFITS data into AIPS and export flagged
    data back to UVFITS.
- name: AIPS
  role: Astronomical Image Processing Software environment in which the visibility
    data reside and are manipulated.
- name: Python
  role: Implementation language of the SERPent 'out of the box' script driving the
    UVFITS manipulation.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.09.001
    title: 'SERPent: Automated reduction and RFI-mitigation software for e-MERLIN'
  source_content_hash: sha256:42af89b698aea27c251ad198c32114f0296688fb6a164d269a1c3cc02a86feaa
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# uvfits-data-manipulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Loading, reordering, flagging and re-exporting UVFITS interferometric visibility data within the AIPS environment via ParselTongue/Python, as operationalised by SERPent for e-MERLIN L-band (1.3–1.8 GHz) and C-band (4–8 GHz) commissioning datasets.

## When to use

Use when the input is a radio-interferometric visibility dataset in UVFITS format that must be loaded into AIPS, restructured per baseline for RFI flagging (e.g. SumThreshold), and exported back ready for calibration.

## When NOT to use

- The input is not radio interferometric visibility data (SERPent's scope is 'any radio interferometric array data', not single-dish or non-radio data).
- The data are already flagged/calibrated and the goal is further imaging or analysis rather than RFI-mitigation-style flagging and re-export of UVFITS files.

## Inputs

- UVFITS visibility file from a radio interferometric observation (e.g. e-MERLIN L-band 1.3–1.8 GHz or C-band 4–8 GHz commissioning data)
- AIPS-hosted visibility data accessed through ParselTongue

## Outputs

- Flagged UVFITS visibility dataset, immediately ready for AIPS calibration
- Per-sample RFI flags propagated to visibility records in the AIPS data area

## How to apply

Load the UVFITS visibility file into AIPS using ParselTongue FIT-loading. Read the visibility amplitudes and reorder them into a time-by-frequency sequence for each interferometer baseline, the arrangement required by the SumThreshold formulation of Offringa et al. 2010 (MNRAS, 405, 155-167). Apply the RFI detection/flagging (e.g. SumThreshold per baseline, plus antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans for e-MERLIN) and write the flags back to the corresponding visibility records in the AIPS data area. Export the flagged dataset back to a UVFITS file via ParselTongue so the output is immediately ready for AIPS calibration. The rationale is that ParselTongue/Python interaction with AIPS lets the pipeline operate 'out of the box' without compilers while preserving AIPS-compatible visibility data structures.

## Related tools

- **SERPent** (Automated reduction and RFI-mitigation pipeline that performs the UVFITS load, per-baseline reordering, flagging and re-export.) — daniellefenech/SERPent
- **Parseltongue** (Language used by SERPent to load UVFITS data into AIPS and export flagged data back to UVFITS.)
- **AIPS** (Astronomical Image Processing Software environment in which the visibility data reside and are manipulated.)
- **Python** (Implementation language of the SERPent 'out of the box' script driving the UVFITS manipulation.)

## Evaluation signals

- The flagged output file exists and is a valid UVFITS file readable back into AIPS.
- Flagged data fraction is consistent with reported RFI levels: <20%–25% for problematic L-band observations and <5% for the quieter C-band.
- Output is loadable and 'immediately ready for AIPS calibration' without further format conversion.
- Flags correspond one-to-one with the visibility records detected as RFI-affected, including antenna zero-amplitude dropouts where present.

## Limitations

- Flagging rate is 110 GB day−1 on a 'high-end' computer (16 CPUs, 100 GB memory), ∼6.9 GB CPU−1 day−1 — large datasets take substantial time.
- Extra flagging behaviours (antenna zero-amplitude dropouts, Lovell telescope phase calibrator stationary scans) are specific to the e-MERLIN system.
- The article's discussion section content is missing, so extended caveats could not be traced.

## Evidence
