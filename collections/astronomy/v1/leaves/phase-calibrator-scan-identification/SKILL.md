---
name: phase-calibrator-scan-identification
description: Use when reducing e-MERLIN interferometric visibility data whose phase-calibrator
  observations include stationary scans from the Lovell telescope, so these system-inherent
  scans are flagged before AIPS calibration.
when_to_use_negative:
- Data are not from e-MERLIN or do not contain Lovell telescope phase calibrator stationary
  scans — the filter targets a signature inherent to the e-MERLIN system.
- The visibility data have not yet passed through SumThreshold RFI flagging and antenna
  zero-amplitude dropout flagging, so the input flag table does not reflect the full
  SERPent flagging chain.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3321
tools:
- name: SERPent
  role: Automated reduction and RFI-mitigation pipeline containing the Lovell phase-calibrator
    stationary-scan flagging step
  repo: daniellefenech/SERPent
- name: Parseltongue
  role: Language in which SERPent is written, enabling interaction with AIPS to apply
    the flagging step
- name: AIPS
  role: Environment holding the visibility data; the flagged output is immediately
    ready for AIPS calibration
- name: Python
  role: Language of the SERPent 'out of the box' script that executes the flagging
    workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.09.001
    title: 'SERPent: Automated reduction and RFI-mitigation software for e-MERLIN'
  source_content_hash: sha256:0a364c15da16d9500f2505d75f267e7cb7b796d899c75d138d939b2afc225e12
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3321
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# phase-calibrator-scan-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A flagging step within the SERPent e-MERLIN pipeline that identifies and flags Lovell telescope phase calibrator stationary scans, an inherent artefact of the e-MERLIN system, in addition to RFI-affected visibilities and antenna zero-amplitude dropouts.

## When to use

Use when reducing e-MERLIN interferometric visibility data whose phase-calibrator observations include stationary scans from the Lovell telescope, so these system-inherent scans are flagged before AIPS calibration.

## When NOT to use

- Data are not from e-MERLIN or do not contain Lovell telescope phase calibrator stationary scans — the filter targets a signature inherent to the e-MERLIN system.
- The visibility data have not yet passed through SumThreshold RFI flagging and antenna zero-amplitude dropout flagging, so the input flag table does not reflect the full SERPent flagging chain.

## Inputs

- e-MERLIN visibility/flag output from prior SumThreshold RFI flagging and antenna zero-amplitude dropout flagging steps (AIPS UVFITS data)
- e-MERLIN commissioning dataset scan metadata identifying Lovell telescope phase calibrator scans (L-band 1.3–1.8 GHz or C-band 4–8 GHz)

## Outputs

- flagged_visibilities.uvf with Lovell phase-calibrator stationary scans marked flagged, ready for AIPS calibration

## How to apply

Load the visibility/flag output from the earlier SumThreshold RFI flagging and antenna zero-amplitude dropout flagging steps into AIPS via the Parseltongue interface used by SERPent. Using scan metadata from the e-MERLIN commissioning dataset, identify scans observing the Lovell telescope phase calibrator and apply the discrete filter that detects the stationary-scan signature characteristic of these observations. Mark the affected phase-calibrator stationary scans as flagged in the visibility/flag table while leaving all prior RFI and dropout flags untouched, then write the updated data to flagged_visibilities.uvf. The rationale is that these stationary scans are inherent to the e-MERLIN system and, like RFI, must be removed so that the output is immediately ready for AIPS calibration.

## Related tools

- **SERPent** (Automated reduction and RFI-mitigation pipeline containing the Lovell phase-calibrator stationary-scan flagging step) — daniellefenech/SERPent
- **Parseltongue** (Language in which SERPent is written, enabling interaction with AIPS to apply the flagging step)
- **AIPS** (Environment holding the visibility data; the flagged output is immediately ready for AIPS calibration)
- **Python** (Language of the SERPent 'out of the box' script that executes the flagging workflow)

## Evaluation signals

- Lovell phase-calibrator stationary scans are marked flagged in the visibility/flag table while all other RFI and antenna zero-amplitude dropout flags from earlier steps remain untouched.
- The updated output is written as flagged_visibilities.uvf.
- The flagged output is immediately ready for AIPS calibration without further manual flagging.
- The flagging level is more accurate and delicate than visual manual flagging, per the article's reported behaviour.

## Limitations

- The stationary-scan signature is specific to the e-MERLIN system and the Lovell telescope phase calibrator; the filter may not generalise to other arrays' systematics.
- The step depends on correct scan metadata to identify phase-calibrator observations in the commissioning dataset.
- No changelog was found for the repository, and discussion-section content is missing, so implementation details beyond the abstract-level description are unavailable.

## Evidence
