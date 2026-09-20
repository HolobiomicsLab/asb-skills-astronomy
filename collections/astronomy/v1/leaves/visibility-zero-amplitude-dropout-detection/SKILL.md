---
name: visibility-zero-amplitude-dropout-detection
description: Use when reducing e-MERLIN visibility data (L-band 1.3–1.8 GHz or C-band
  4–8 GHz) where antenna amplitudes may drop to exactly zero, and these dropout records
  must be flagged before AIPS calibration.
when_to_use_negative:
- Data are from an interferometric array other than e-MERLIN where zero-amplitude
  dropouts are not an inherent system defect (though SERPent itself claims suitability
  for any radio interferometric array data, this specific defect signature is e-MERLIN-inherent)
- The visibility data have not yet passed through the SumThreshold RFI-flagging step,
  so flag provenance between RFI flags and dropout flags cannot be separated
- The task is generic RFI detection (e.g. threshold-based or morphological detection
  per Offringa et al. 2010/2012) rather than detection of the specific zero-amplitude
  dropout signature
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_3321
tools:
- name: SERPent
  role: Automated pipeline in which the zero-amplitude dropout flagging runs alongside
    SumThreshold RFI flagging
  repo: daniellefenech/SERPent
- name: ParselTongue
  role: Language in which SERPent is written, enabling access to AIPS visibility data
    for the dropout scan
  repo: daniellefenech/SERPent
- name: AIPS
  role: Host environment for the visibility data; flagged output is immediately ready
    for AIPS calibration
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.09.001
    title: 'SERPent: Automated reduction and RFI-mitigation software for e-MERLIN'
  source_content_hash: sha256:2a899b82b2b84e388704a7d42596b2184b618d825d786b92c1f7755b5d5e1f31
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
  - ParselTongue
  - Python
  - SERPent
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_3321
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# visibility-zero-amplitude-dropout-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects and flags antenna zero-amplitude dropouts — an instrumental defect inherent to the e-MERLIN system — in visibility data alongside the SumThreshold-based RFI flagging performed by the SERPent pipeline.

## When to use

Use when reducing e-MERLIN visibility data (L-band 1.3–1.8 GHz or C-band 4–8 GHz) where antenna amplitudes may drop to exactly zero, and these dropout records must be flagged before AIPS calibration.

## When NOT to use

- Data are from an interferometric array other than e-MERLIN where zero-amplitude dropouts are not an inherent system defect (though SERPent itself claims suitability for any radio interferometric array data, this specific defect signature is e-MERLIN-inherent)
- The visibility data have not yet passed through the SumThreshold RFI-flagging step, so flag provenance between RFI flags and dropout flags cannot be separated
- The task is generic RFI detection (e.g. threshold-based or morphological detection per Offringa et al. 2010/2012) rather than detection of the specific zero-amplitude dropout signature

## Inputs

- Visibility dataset with flag column as produced by the SumThreshold RFI-flagging step in SERPent (ParselTongue/AIPS visibility format)
- e-MERLIN commissioning or observation data (L-band 1.3–1.8 GHz or C-band 4–8 GHz)

## Outputs

- Visibility/flag table with antenna zero-amplitude dropout records additionally flagged (flags preserved from SumThreshold step)
- Optional per-record flag listing (e.g. dropout_flags.csv with antenna, time, channel, flag state)

## How to apply

Within the SERPent pipeline (written in ParselTongue/Python operating on AIPS data), run this dropout flagging step alongside the SumThreshold RFI flagging rather than as a manual visual inspection. Load the visibility dataset and its flag column, then scan each antenna's amplitudes over time and frequency to identify intervals where the recorded amplitude is exactly zero — the signature of e-MERLIN antenna zero-amplitude dropouts. Mark the affected visibility records as flagged without overwriting flags already set by the SumThreshold step, since the two defect types (RFI contamination vs. instrumental dropout) must both be captured. The flagged output should be immediately ready for AIPS calibration. Note that SERPent also flags Lovell telescope phase calibrator stationary scans as a separate e-MERLIN-inherent defect, so confirm which defect class each flag corresponds to.

## Related tools

- **SERPent** (Automated pipeline in which the zero-amplitude dropout flagging runs alongside SumThreshold RFI flagging) — daniellefenech/SERPent
- **ParselTongue** (Language in which SERPent is written, enabling access to AIPS visibility data for the dropout scan) — daniellefenech/SERPent
- **AIPS** (Host environment for the visibility data; flagged output is immediately ready for AIPS calibration)

## Evaluation signals

- Every flagged dropout record has visibility amplitude exactly zero at the flagged time/channel/antenna
- Flags already set by the SumThreshold step are unchanged after the dropout flagging pass (no flag overwrites)
- Dropout flags are attributable to e-MERLIN-inherent defects, distinct from RFI-affected visibilities flagged by SumThreshold
- Output visibility/flag table loads into AIPS and is accepted by the calibration steps without format errors
- No unflagged zero-amplitude intervals remain when rescanning antenna amplitudes over time and frequency

## Limitations

- The zero-amplitude dropout defect is specific to the e-MERLIN system; the signature (exactly zero amplitude) may not generalize to other arrays
- SERPent also flags Lovell telescope phase calibrator stationary scans as a separate defect, so dropout flags must not be conflated with that flag class
- Flagging rate of the overall pipeline is ~110 GB day−1 on a 16-CPU, 100 GB machine (~6.9 GB CPU−1 day−1), which constrains throughput for large datasets

## Evidence
