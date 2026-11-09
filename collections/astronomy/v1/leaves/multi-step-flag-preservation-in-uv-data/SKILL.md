---
name: multi-step-flag-preservation-in-uv-data
description: Use when reducing e-MERLIN interferometric visibility data where Lovell
  phase-calibrator stationary scans must be flagged after RFI and dropout flagging,
  with prior flags preserved.
when_to_use_negative:
- Input is a raw, unflagged uv dataset that has not yet been through SumThreshold
  RFI flagging and antenna zero-amplitude dropout flagging — this step is an additional,
  later stage in the SERPent sequence
- Data are from an array without Lovell telescope phase calibrator stationary scans,
  or are non-interferometric; SERPent targets radio interferometric array data and
  its Lovell-specific filter is e-MERLIN inherent behavior
- Goal is calibration or imaging rather than RFI/stationary-scan flagging of visibilities
tools:
- name: SERPent
  role: Automated reduction and RFI-mitigation pipeline that performs the Lovell phase
    calibrator stationary-scan flagging step alongside SumThreshold RFI and dropout
    flagging
  repo: https://github.com/daniellefenech/SERPent
- name: Parseltongue
  role: Language in which SERPent is written, enabling scripted interaction with the
    AIPS environment to access and update flag tables
- name: AIPS
  role: Astronomical Image Processing Software environment in which the flagged visibility
    data are held and into which the output is ready for calibration
- name: Python
  role: Scripting language component of the SERPent 'out of the box' script that executes
    the flagging steps
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.09.001
    title: 'SERPent: Automated reduction and RFI-mitigation software for e-MERLIN'
  source_content_hash: sha256:197463502881b03e11645c3d42a73acb0cd3023d09771b72b6e333c78f49e12f
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Multi-step flag preservation in uv data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A discrete filtering step in the SERPent pipeline that flags Lovell telescope phase calibrator stationary scans — inherent to the e-MERLIN system — on top of SumThreshold RFI flagging and antenna zero-amplitude dropout flagging, without disturbing the existing flag table entries.

## When to use

Use when reducing e-MERLIN interferometric visibility data where Lovell phase-calibrator stationary scans must be flagged after RFI and dropout flagging, with prior flags preserved.

## When NOT to use

- Input is a raw, unflagged uv dataset that has not yet been through SumThreshold RFI flagging and antenna zero-amplitude dropout flagging — this step is an additional, later stage in the SERPent sequence
- Data are from an array without Lovell telescope phase calibrator stationary scans, or are non-interferometric; SERPent targets radio interferometric array data and its Lovell-specific filter is e-MERLIN inherent behavior
- Goal is calibration or imaging rather than RFI/stationary-scan flagging of visibilities

## Inputs

- Flagged visibility data output from SumThreshold RFI flagging and antenna zero-amplitude dropout flagging (e-MERLIN commissioning dataset, L-band 1.3–1.8 GHz or C-band 4–8 GHz)
- Scan metadata identifying Lovell telescope phase calibrator observations

## Outputs

- Flagged visibility data written to flagged_visibilities.uvf, with Lovell phase-calibrator stationary scans flagged and all prior RFI/dropout flags intact
- Output immediately ready for AIPS calibration

## How to apply

Load the visibility/flag output of the prior SumThreshold RFI flagging and antenna zero-amplitude dropout flagging steps into AIPS via the Parseltongue interface used by SERPent. Identify scans observing the Lovell telescope phase calibrator using the e-MERLIN commissioning dataset scan metadata, then apply the discrete filter that detects the stationary-scan signature characteristic of Lovell phase calibrator observations inherent to the e-MERLIN system. Mark only those stationary scans as flagged in the flag table, leaving all other RFI and dropout flags from earlier steps untouched. Write the updated flagged visibility data to flagged_visibilities.uvf, which is immediately ready for AIPS calibration.

## Related tools

- **SERPent** (Automated reduction and RFI-mitigation pipeline that performs the Lovell phase calibrator stationary-scan flagging step alongside SumThreshold RFI and dropout flagging) — https://github.com/daniellefenech/SERPent
- **Parseltongue** (Language in which SERPent is written, enabling scripted interaction with the AIPS environment to access and update flag tables)
- **AIPS** (Astronomical Image Processing Software environment in which the flagged visibility data are held and into which the output is ready for calibration)
- **Python** (Scripting language component of the SERPent 'out of the box' script that executes the flagging steps)

## Evaluation signals

- The output file flagged_visibilities.uvf is produced containing the updated flag table
- Only Lovell phase calibrator stationary scans are newly marked flagged; all RFI and zero-amplitude dropout flags from earlier steps are unchanged (flag-preservation invariant)
- The flagged output is immediately ready for AIPS calibration without manual flag cleanup, as reported for SERPent output
- Detection/flagging accuracy remains at least as delicate as visual manual flagging, per the article's stated comparison

## Limitations

- The Lovell phase calibrator stationary-scan signature is specific to the e-MERLIN system; the filter targets a systematic inherent to that array rather than generic RFI
- This step depends on prior SumThreshold and dropout flagging steps having run; it is not a standalone RFI mitigation
- Flagging rate is throughput-limited: 110 GB day−1 on a 'high-end' computer (16 CPUs, 100 GB memory), ∼6.9 GB CPU−1 day−1

## Evidence
