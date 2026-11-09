---
name: radio-interferometric-visibility-flagging
description: Use when raw radio interferometric visibility data (e.g. e-MERLIN L-band
  1.3–1.8 GHz or C-band 4–8 GHz observations) contain RFI-affected visibilities, antenna
  zero-amplitude dropouts, or Lovell phase-calibrator stationary scans that must be
  flagged automatically before calibration.
when_to_use_negative:
- Input is not radio interferometric visibility data — although SERPent is stated
  to be suitable for use with any radio interferometric array data, it targets e-MERLIN
  system artifacts (zero-amplitude dropouts, Lovell stationary scans) that do not
  exist in other observatory formats
- Data have already been manually or automatically flagged and only require calibration
  rather than re-flagging
- Human review of marginal RFI detections is required — SERPent's automated detection
  is designed to be more accurate and delicate than visual manual flagging, not a
  replacement for expert inspection of ambiguous cases
edam_operation: http://edamontology.org/operation_3695
tools:
- name: SERPent
  role: The automated reduction and RFI-mitigation pipeline that performs SumThreshold
    RFI flagging plus e-MERLIN-specific dropout and stationary-scan flagging
  repo: daniellefenech/SERPent
- name: Parseltongue
  role: Scripting language SERPent is written in, enabling interaction with AIPS and
    direct manipulation of visibility/flag data
- name: AIPS
  role: Astronomical Image Processing Software environment hosting the visibility
    data; flagged output is immediately ready for AIPS calibration
- name: Python
  role: Implementation language of the SERPent 'out of the box' script
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.09.001
    title: 'SERPent: Automated reduction and RFI-mitigation software for e-MERLIN'
  source_content_hash: sha256:9d5e64de71ff982efe0fe2182f3b94420ceb979ebb37086a26ac16ae753550e8
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
yes_rate: 0.75
n_claims: 8
---

# radio-interferometric-visibility-flagging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated visibility flagging for radio interferometric data following the SERPent pipeline, which applies SumThreshold-based RFI mitigation plus e-MERLIN-specific flagging of antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans, producing output immediately ready for AIPS calibration.

## When to use

Use when raw radio interferometric visibility data (e.g. e-MERLIN L-band 1.3–1.8 GHz or C-band 4–8 GHz observations) contain RFI-affected visibilities, antenna zero-amplitude dropouts, or Lovell phase-calibrator stationary scans that must be flagged automatically before calibration.

## When NOT to use

- Input is not radio interferometric visibility data — although SERPent is stated to be suitable for use with any radio interferometric array data, it targets e-MERLIN system artifacts (zero-amplitude dropouts, Lovell stationary scans) that do not exist in other observatory formats
- Data have already been manually or automatically flagged and only require calibration rather than re-flagging
- Human review of marginal RFI detections is required — SERPent's automated detection is designed to be more accurate and delicate than visual manual flagging, not a replacement for expert inspection of ambiguous cases

## Inputs

- Raw or partially reduced radio interferometric visibility dataset (AIPS visibility format accessed via Parseltongue), e.g. e-MERLIN commissioning data
- Visibility/flag table output from the SumThreshold RFI-flagging step for subsequent dropout and stationary-scan flagging

## Outputs

- Flagged visibility data with RFI-affected visibilities, antenna zero-amplitude dropouts, and Lovell phase-calibrator stationary scans marked as flagged
- Flag table with per-record flag states (e.g. dropout_flags.csv with antenna, time, channel, and flag state columns, per source-card workflow)
- Calibration-ready flagged dataset (e.g. flagged_visibilities.uvf per source-card workflow)

## How to apply

Run SERPent, a ParselTongue/Python script operating on visibility data in the AIPS environment, to apply the SumThreshold methodology (Offringa et al. 2010a) for detecting and flagging RFI-affected visibilities. Alongside SumThreshold RFI flagging, detect intervals where an antenna's recorded visibility amplitude is exactly zero (e-MERLIN antenna zero-amplitude dropouts) and mark those records as flagged without altering flags set by the RFI step. Additionally apply the discrete filter that identifies the stationary-scan signature of Lovell telescope phase calibrator observations, an artifact inherent to the e-MERLIN system, and flag those scans while leaving prior flags intact. Decision rationale: with increased e-MERLIN bandwidth, RFI detection rates have dramatically increased, particularly at L-band (1.4–1.8 GHz) where RFI typically amounts to <20%–25% versus <5% for C-band, so automated flagging makes maximum use of available bandwidth. Expect a flagging rate of ~110 GB day−1 (~6.9 GB CPU−1 day−1 on a 16-CPU, 100 GB memory machine), with the flagged output immediately ready for AIPS calibration.

## Related tools

- **SERPent** (The automated reduction and RFI-mitigation pipeline that performs SumThreshold RFI flagging plus e-MERLIN-specific dropout and stationary-scan flagging) — daniellefenech/SERPent
- **Parseltongue** (Scripting language SERPent is written in, enabling interaction with AIPS and direct manipulation of visibility/flag data)
- **AIPS** (Astronomical Image Processing Software environment hosting the visibility data; flagged output is immediately ready for AIPS calibration)
- **Python** (Implementation language of the SERPent 'out of the box' script)

## Evaluation signals

- Flagged output loads directly into AIPS calibration without intermediate cleaning, matching the reported 'immediately ready for AIPS calibration' property
- RFI flag fractions are consistent with expected ranges: <20%–25% for problematic L-band (1.3–1.8 GHz) data and <5% for C-band (4–8 GHz) data
- Antenna zero-amplitude dropout flags mark records where visibility amplitude is exactly zero, and do not overwrite flags already set by the SumThreshold step
- Lovell phase-calibrator stationary scans are flagged while all other RFI and dropout flags from earlier steps remain untouched
- Throughput is in the reported range of ~110 GB day−1 (~6.9 GB CPU−1 day−1) on a high-end 16-CPU, 100 GB memory machine

## Limitations

- The most aggressive flagging behavior targets L-band (1.4–1.8 GHz) where the spectrum is actively used for other purposes and RFI typically amounts to <20%–25%; C-band data are generally RFI-quiet (<5%)
- The e-MERLIN-specific flagging steps (zero-amplitude dropouts, Lovell phase calibrator stationary scans) are tied to defects inherent to the e-MERLIN system
- Flagging rate is bounded at ~110 GB day−1 on a 'high-end' computer (16 CPUs, 100 GB memory)
- The discussion section content of the paper is missing from the available context, and no changelog was found

## Evidence
