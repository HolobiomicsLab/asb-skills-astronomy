---
name: e-merlin-instrumental-artifact-mitigation
description: Use when reducing e-MERLIN visibility data in AIPS/ParselTongue and Lovell
  telescope phase calibrator stationary scans (an inherent e-MERLIN instrumental artifact)
  must be flagged alongside RFI and zero-amplitude dropouts.
when_to_use_negative:
- Data are not from e-MERLIN (or an array with a Lovell telescope phase calibrator
  stationary-scan artifact) — SERPent's instrumental flagging targets scans inherent
  to the e-MERLIN system
- Data have not yet passed through the SumThreshold RFI flagging and antenna zero-amplitude
  dropout flagging steps of the pipeline
- Data are already calibrated or already flagged for stationary scans and only require
  AIPS calibration
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SERPent
  role: Automated pipeline implementing the flagging of Lovell telescope phase calibrator
    stationary scans
  repo: https://github.com/daniellefenech/SERPent
- name: ParselTongue
  role: Language in which SERPent is written, enabling programmatic interaction with
    AIPS visibility data
- name: AIPS
  role: Environment holding the visibility/flag tables; flagged output is immediately
    ready for AIPS calibration
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.09.001
    title: 'SERPent: Automated reduction and RFI-mitigation software for e-MERLIN'
  source_content_hash: sha256:46a0863d5dc9f2f30b206eaa136262e4b80fe7fb53aa1c9448e7bdc59a158acc
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# e-merlin-instrumental-artifact-mitigation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A SERPent pipeline step that flags Lovell telescope phase calibrator stationary scans — an instrumental artifact inherent to the e-MERLIN system — in addition to RFI-affected visibilities and antenna zero-amplitude dropouts, producing flagged data immediately ready for AIPS calibration.

## When to use

Use when reducing e-MERLIN visibility data in AIPS/ParselTongue and Lovell telescope phase calibrator stationary scans (an inherent e-MERLIN instrumental artifact) must be flagged alongside RFI and zero-amplitude dropouts.

## When NOT to use

- Data are not from e-MERLIN (or an array with a Lovell telescope phase calibrator stationary-scan artifact) — SERPent's instrumental flagging targets scans inherent to the e-MERLIN system
- Data have not yet passed through the SumThreshold RFI flagging and antenna zero-amplitude dropout flagging steps of the pipeline
- Data are already calibrated or already flagged for stationary scans and only require AIPS calibration

## Inputs

- Visibility/flag output from prior SERPent SumThreshold RFI flagging and antenna zero-amplitude dropout flagging steps (AIPS visibility data)
- e-MERLIN scan metadata identifying phase calibrator observations (L-band 1.3–1.8 GHz or C-band 4–8 GHz commissioning data)

## Outputs

- Flagged visibility data (flagged_visibilities.uvf) with Lovell phase calibrator stationary scans marked flagged
- Updated AIPS flag table preserving all prior RFI and dropout flags

## How to apply

Load the visibility/flag output from the prior SumThreshold RFI flagging and antenna zero-amplitude dropout flagging steps into AIPS via the ParselTongue interface used by SERPent. Identify scans observing the Lovell telescope phase calibrator using the scan metadata of the e-MERLIN commissioning dataset, then apply the discrete detection step that flags the stationary-scan signature inherent to the e-MERLIN system. Mark only the affected phase-calibrator stationary scans as flagged in the visibility/flag table, leaving all earlier RFI and dropout flags untouched, and write the updated data to flagged_visibilities.uvf. The rationale is that these stationary scans are instrumental artifacts of the e-MERLIN system rather than RFI, so they must be removed by a dedicated flagging step rather than by the SumThreshold RFI classifier.

## Related tools

- **SERPent** (Automated pipeline implementing the flagging of Lovell telescope phase calibrator stationary scans) — https://github.com/daniellefenech/SERPent
- **ParselTongue** (Language in which SERPent is written, enabling programmatic interaction with AIPS visibility data)
- **AIPS** (Environment holding the visibility/flag tables; flagged output is immediately ready for AIPS calibration)

## Evaluation signals

- Lovell telescope phase calibrator stationary scans are marked flagged in the output flag table
- All pre-existing RFI flags (SumThreshold) and antenna zero-amplitude dropout flags remain unchanged in the flag table
- Output file flagged_visibilities.uvf is written and loadable in AIPS
- Flagged output is immediately ready for AIPS calibration without further manual flagging
- Detection/flagging is more accurate and delicate than visual manual flagging, per the article

## Limitations

- The stationary-scan artifact is specific to the e-MERLIN system (Lovell telescope phase calibrator); the step is not meaningful for other arrays' data
- Discussion section content is missing from the indexed article, so caveats from that section are unavailable
- Flagging throughput is bounded at ~110 GB/day (∼6.9 GB CPU−1 day−1) on a 16-CPU, 100 GB memory machine

## Evidence

- [abstract] the script also flags antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans inherent to the e-MERLIN system: "the script also flags antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans inherent to the e-MERLIN system"
- [abstract] The level of RFI detection and flagging is more accurate and delicate than visual manual flagging, with the output immediately ready for AIPS calibration: "The level of RFI detection and flagging is more accurate and delicate than visual manual flagging, with the output immediately ready for AIPS calibration"
- [readme] SERPent is written in ParselTongue and Python to interact directly with data in the AIPS environment: "The SERPent software is written in ParselTongue and Python to interact directly with data in the AIPS environment."
