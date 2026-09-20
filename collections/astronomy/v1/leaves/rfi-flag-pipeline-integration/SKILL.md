---
name: rfi-flag-pipeline-integration
description: Use when reducing e-MERLIN interferometric visibility data where, beyond
  RFI flagged by SumThreshold, antenna zero-amplitude dropouts or Lovell phase calibrator
  stationary scans must also be flagged before AIPS calibration.
when_to_use_negative:
- Input is already calibrated or has passed through AIPS calibration — this skill
  flags raw/early-stage visibility defects before calibration
- Data are from an array whose defects are not e-MERLIN antenna zero-amplitude dropouts
  or Lovell stationary scans and need no such flagging
- Only RFI mitigation is required and SumThreshold flagging alone (without the e-MERLIN
  system-defect flagging) suffices
edam_operation: http://edamontology.org/operation_3695
tools:
- name: SERPent
  role: Automated reduction and RFI-mitigation pipeline whose additional flagging
    step flags antenna zero-amplitude dropouts and Lovell stationary scans
  repo: https://github.com/daniellefenech/SERPent
- name: Parseltongue
  role: Language in which SERPent is written, enabling interaction with AIPS visibility
    data
- name: AIPS
  role: Environment holding the visibility data being flagged; the flagged output
    is immediately ready for AIPS calibration
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.09.001
    title: 'SERPent: Automated reduction and RFI-mitigation software for e-MERLIN'
  source_content_hash: sha256:0f3dce0e7a958822a9699721b3c78d44376217e524e12639217d619a4791d52a
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
yes_rate: 0.9230769230769231
n_claims: 13
---

# rfi-flag-pipeline-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An automated e-MERLIN visibility flagging step within the SERPent pipeline that, in addition to SumThreshold-based RFI flagging, flags antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans — defects inherent to the e-MERLIN system.

## When to use

Use when reducing e-MERLIN interferometric visibility data where, beyond RFI flagged by SumThreshold, antenna zero-amplitude dropouts or Lovell phase calibrator stationary scans must also be flagged before AIPS calibration.

## When NOT to use

- Input is already calibrated or has passed through AIPS calibration — this skill flags raw/early-stage visibility defects before calibration
- Data are from an array whose defects are not e-MERLIN antenna zero-amplitude dropouts or Lovell stationary scans and need no such flagging
- Only RFI mitigation is required and SumThreshold flagging alone (without the e-MERLIN system-defect flagging) suffices

## Inputs

- e-MERLIN visibility datasets (L-band 1.3–1.8 GHz or C-band 4–8 GHz commissioning data) in AIPS format
- Flag column as produced by the SumThreshold RFI-flagging step within SERPent

## Outputs

- Flagged visibility dataset with RFI, antenna zero-amplitude dropout, and Lovell stationary-scan flags applied, ready for AIPS calibration
- Flag table of flagged records (e.g. dropout_flags.csv) with antenna, time, channel, and flag state columns

## How to apply

Run SERPent (a ParselTongue/Python script operating in the AIPS environment) on the visibility dataset: the pipeline applies the SumThreshold methodology (Offringa et al., 2010a) to flag RFI-affected visibilities, then additionally scans antenna visibility amplitudes to flag zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans inherent to the e-MERLIN system. Preserve flags already set by the SumThreshold step when adding the system-defect flags. Write the flagged visibility/flag records out (e.g. one row per flagged record identifying antenna, time, channel, and flag state, such as dropout_flags.csv). The rationale is that these e-MERLIN-specific defects are not captured by SumThreshold's morphological RFI detection alone, and the output must be immediately ready for AIPS calibration.

## Related tools

- **SERPent** (Automated reduction and RFI-mitigation pipeline whose additional flagging step flags antenna zero-amplitude dropouts and Lovell stationary scans) — https://github.com/daniellefenech/SERPent
- **Parseltongue** (Language in which SERPent is written, enabling interaction with AIPS visibility data)
- **AIPS** (Environment holding the visibility data being flagged; the flagged output is immediately ready for AIPS calibration)

## Evaluation signals

- Flagged output loads correctly in AIPS and is accepted as immediately ready for AIPS calibration
- Zero-amplitude intervals in each antenna's visibility amplitudes over time and frequency are marked flagged, while flags already set by SumThreshold remain unchanged
- Lovell telescope phase calibrator stationary scans are flagged alongside RFI and dropout flags
- Flag table records include identifying columns (antenna, time, channel, flag state) with one row per flagged record
- RFI fraction removed is consistent with reported levels (<20%–25% for problematic L-band, <5% for quieter C-band observations)

## Limitations

- The Lovell telescope phase calibrator stationary scans and zero-amplitude dropouts are defects inherent to the e-MERLIN system, so this flagging step is specific to e-MERLIN data
- Flagging rate is limited to 110 GB day−1 on a high-end computer (16 CPUs, 100 GB memory), ∼6.9 GB CPU−1 day−1

## Evidence
