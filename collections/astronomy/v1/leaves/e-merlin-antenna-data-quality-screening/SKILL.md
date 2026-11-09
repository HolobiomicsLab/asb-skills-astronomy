---
name: e-merlin-antenna-data-quality-screening
description: Use when reducing e-MERLIN visibility datasets where, beyond RFI flagged
  by SumThreshold, antenna zero-amplitude dropouts or Lovell telescope phase calibrator
  stationary scans must be identified and flagged before calibration.
when_to_use_negative:
- Input visibility data is not from a radio interferometric pipeline where antenna
  zero-amplitude dropouts are a known defect — SERPent's SumThreshold-based RFI flagging
  alone may be the appropriate step instead
- Data has already passed through SERPent and dropouts have been flagged; re-running
  would duplicate the screening step
- The goal is generic RFI detection rather than e-MERLIN-inherent defect screening;
  the Offringa et al. (2010, 2012) SumThreshold algorithms address RFI, not antenna
  dropouts
edam_operation: http://edamontology.org/operation_0282
tools:
- name: SERPent
  role: Automated reduction and RFI-mitigation pipeline whose additional flagging
    step flags antenna zero-amplitude dropouts and Lovell telescope phase calibrator
    stationary scans alongside SumThreshold RFI flagging
  repo: https://github.com/daniellefenech/SERPent
- name: ParselTongue
  role: Language in which SERPent is written, enabling direct interaction with visibility
    data in the AIPS environment
- name: AIPS
  role: Astronomical Image Processing Software environment in which the visibility/flag
    data are manipulated and into which the flagged output feeds for calibration
- name: Python
  role: Implementation language of the SERPent 'out of the box' script
- name: e-MERLIN
  role: Instrument whose inherent visibility defects (antenna zero-amplitude dropouts,
    Lovell phase calibrator stationary scans) this screening targets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.09.001
    title: 'SERPent: Automated reduction and RFI-mitigation software for e-MERLIN'
  source_content_hash: sha256:817dd7926dfcce8813f9d8849afb9be3e2bddd5d4e4e6e7c762404bb4a0dfd71
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
  - e-MERLIN
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# e-merlin-antenna-data-quality-screening

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An auxiliary flagging step of the SERPent pipeline that, in addition to SumThreshold-based RFI flagging, detects and flags antenna zero-amplitude dropouts (and Lovell telescope phase calibrator stationary scans) — visibility defects inherent to the e-MERLIN system — so the output is immediately ready for AIPS calibration.

## When to use

Use when reducing e-MERLIN visibility datasets where, beyond RFI flagged by SumThreshold, antenna zero-amplitude dropouts or Lovell telescope phase calibrator stationary scans must be identified and flagged before calibration.

## When NOT to use

- Input visibility data is not from a radio interferometric pipeline where antenna zero-amplitude dropouts are a known defect — SERPent's SumThreshold-based RFI flagging alone may be the appropriate step instead
- Data has already passed through SERPent and dropouts have been flagged; re-running would duplicate the screening step
- The goal is generic RFI detection rather than e-MERLIN-inherent defect screening; the Offringa et al. (2010, 2012) SumThreshold algorithms address RFI, not antenna dropouts

## Inputs

- e-MERLIN visibility dataset with flag column (ParselTongue/AIPS format), e.g. from L-band (1.3–1.8 GHz) or C-band (4–8 GHz) commissioning observations
- Flag column produced by the SumThreshold RFI-flagging step within SERPent

## Outputs

- Visibility/flag table with antenna zero-amplitude dropout records (and Lovell phase calibrator stationary scans) additionally marked as flagged, preserving SumThreshold flags
- Dropout-flagged records table (e.g. dropout_flags.csv with antenna, time, channel, flag state) ready for AIPS calibration

## How to apply

Run SERPent (a ParselTongue/Python script operating on data in the AIPS environment) on the e-MERLIN visibility dataset after or alongside the SumThreshold RFI-flagging step. Scan each antenna's visibility amplitudes over time and frequency for intervals where the recorded amplitude is exactly zero, which is the signature of e-MERLIN antenna zero-amplitude dropouts; also flag Lovell telescope phase calibrator stationary scans, which are inherent to the e-MERLIN system. Mark the affected visibility records as flagged in the flag column without overwriting flags already set by the SumThreshold step. The rationale is that these defects are instrumental to e-MERLIN and are not caught by RFI classification algorithms (Offringa et al. 2010, 2012) alone, so a dedicated screening pass is required to make maximum use of the available bandwidth and produce output immediately ready for AIPS calibration.

## Related tools

- **SERPent** (Automated reduction and RFI-mitigation pipeline whose additional flagging step flags antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans alongside SumThreshold RFI flagging) — https://github.com/daniellefenech/SERPent
- **ParselTongue** (Language in which SERPent is written, enabling direct interaction with visibility data in the AIPS environment)
- **AIPS** (Astronomical Image Processing Software environment in which the visibility/flag data are manipulated and into which the flagged output feeds for calibration)
- **Python** (Implementation language of the SERPent 'out of the box' script)
- **e-MERLIN** (Instrument whose inherent visibility defects (antenna zero-amplitude dropouts, Lovell phase calibrator stationary scans) this screening targets)

## Evaluation signals

- All visibility records with exactly zero amplitude on any antenna are present in the flagged set, and each carries a dropout flag
- Flags set by the SumThreshold RFI-flagging step are unchanged after the dropout screening pass (no overwriting of prior flags)
- Lovell telescope phase calibrator stationary scans are flagged in addition to RFI-affected visibilities
- The flagged output loads in AIPS and is immediately ready for calibration without manual re-flagging
- Flagging rate is consistent with the documented throughput of ~110 GB day−1 on a 16-CPU, 100 GB memory machine (∼6.9 GB CPU−1 day−1)

## Limitations

- The dropout and stationary-scan defects are specific to the e-MERLIN system; for other arrays SERPent is generally suitable for any radio interferometric array data but these additional flags may not apply
- The screening supplements, not replaces, SumThreshold-based RFI mitigation (Offringa et al. 2010, 2012 algorithms)
- Throughput is bounded at ~110 GB day−1 on a high-end computer (16 CPUs, 100 GB memory), ∼6.9 GB CPU−1 day−1

## Evidence

- [abstract] additional flagging of zero-amplitude dropouts beyond RFI: "In addition to the flagging of RFI affected visibilities, the script also flags antenna zero-amplitude dropouts"
- [abstract] dropouts and Lovell stationary scans are e-MERLIN-inherent defects: "the script also flags antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans inherent to the e-MERLIN system"
- [abstract] SumThreshold methodology underlying the pipeline's flagging: "an automated reduction and RFI-mitigation procedure utilising the SumThreshold methodology (Offringa et al., 2010a)"
- [abstract] output readiness for AIPS calibration: "with the output immediately ready for AIPS calibration"
- [readme] implementation stack: ParselTongue/Python on AIPS data: "The SERPent software is written in ParselTongue and Python to interact directly with data in the AIPS environment."
