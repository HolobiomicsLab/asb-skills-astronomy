---
name: aips-visibility-data-handling
description: Use when interferometric visibility data (e.g. e-MERLIN UVFITS) must
  be read into AIPS, flagged or reduced programmatically, and written back out ready
  for AIPS calibration.
when_to_use_negative:
- The data are not radio interferometric visibilities or cannot be loaded into the
  AIPS environment (SERPent's ParselTongue/AIPS interaction is the substrate of this
  skill)
- The goal is interactive, visual manual flagging — the paper reports automated detection
  and flagging is more accurate and delicate than visual manual flagging
- A non-AIPS processing environment (e.g. CASA-only workflow) is mandatory, since
  the skill's export is explicitly prepared for AIPS calibration
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: SERPent
  role: 'Automated reduction and RFI-mitigation pipeline that operationalises this
    skill: loads UVFITS into AIPS, applies SumThreshold flagging, and exports the
    result'
  repo: https://github.com/daniellefenech/SERPent
- name: Parseltongue
  role: Language in which SERPent is written, enabling programmatic interaction with
    AIPS visibility data (e.g. FIT-loading and UVFITS export)
- name: AIPS
  role: Astronomical Image Processing Software environment in which the visibility
    data are held, flagged and re-exported for calibration
- name: Python
  role: Language of the 'out of the box' SERPent script, free of compilers, used with
    ParselTongue to manipulate the AIPS data area
- name: e-MERLIN
  role: Instrument whose commissioning datasets (L-band 1.3–1.8 GHz, C-band 4–8 GHz)
    supply the visibility data and the system-specific defects (zero-amplitude dropouts,
    Lovell stationary scans)
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.09.001
    title: 'SERPent: Automated reduction and RFI-mitigation software for e-MERLIN'
  source_content_hash: sha256:0e21db9d9b7bfefc833b9cb81e4ac81ae3f78e124d59582e4ffae15cd5206e29
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
  - e-MERLIN
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# aips-visibility-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A method for loading, manipulating and re-exporting radio interferometric visibility data inside the AIPS environment via ParselTongue/Python, as implemented by the SERPent pipeline for e-MERLIN commissioning datasets (L-band 1.3–1.8 GHz and C-band 4–8 GHz).

## When to use

Use when interferometric visibility data (e.g. e-MERLIN UVFITS) must be read into AIPS, flagged or reduced programmatically, and written back out ready for AIPS calibration.

## When NOT to use

- The data are not radio interferometric visibilities or cannot be loaded into the AIPS environment (SERPent's ParselTongue/AIPS interaction is the substrate of this skill)
- The goal is interactive, visual manual flagging — the paper reports automated detection and flagging is more accurate and delicate than visual manual flagging
- A non-AIPS processing environment (e.g. CASA-only workflow) is mandatory, since the skill's export is explicitly prepared for AIPS calibration

## Inputs

- e-MERLIN commissioning UVFITS visibility dataset (L-band 1.3–1.8 GHz or C-band 4–8 GHz)
- AIPS visibility/flag tables accessed through ParselTongue

## Outputs

- Flagged visibility dataset exported back to a UVFITS file, immediately ready for AIPS calibration
- Updated flag column in the AIPS data area marking RFI-affected samples, antenna zero-amplitude dropouts and Lovell phase calibrator stationary scans

## How to apply

Write ParselTongue (Python) scripts that interact directly with data in the AIPS environment: load the e-MERLIN visibility dataset, read the visibility amplitudes and reorder them into time-by-frequency sequences per interferometer baseline as required by the SumThreshold formulation of Offringa et al. 2010 (MNRAS, 405, 155-167). Apply the RFI classification and detection algorithms from Offringa et al. 2010 and Offringa et al. 2012 (A&A, 539, A95), and propagate detected flags to the AIPS visibility records, including flagging of antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans inherent to the e-MERLIN system. Export the flagged dataset back to a UVFITS file via ParselTongue so the output is immediately ready for AIPS calibration. Note that SERPent is a simple 'out of the box' Python script, free of compilers, and suitable for use with any radio interferometric array data.

## Related tools

- **SERPent** (Automated reduction and RFI-mitigation pipeline that operationalises this skill: loads UVFITS into AIPS, applies SumThreshold flagging, and exports the result) — https://github.com/daniellefenech/SERPent
- **Parseltongue** (Language in which SERPent is written, enabling programmatic interaction with AIPS visibility data (e.g. FIT-loading and UVFITS export))
- **AIPS** (Astronomical Image Processing Software environment in which the visibility data are held, flagged and re-exported for calibration)
- **Python** (Language of the 'out of the box' SERPent script, free of compilers, used with ParselTongue to manipulate the AIPS data area)
- **e-MERLIN** (Instrument whose commissioning datasets (L-band 1.3–1.8 GHz, C-band 4–8 GHz) supply the visibility data and the system-specific defects (zero-amplitude dropouts, Lovell stationary scans))

## Evaluation signals

- The exported output file exists and is a valid UVFITS file loadable in AIPS
- The flagged data fraction is consistent with the paper's reported RFI levels: <20%–25% for problematic L-band observations and <5% for the generally RFI quieter C-band
- Flags already set by the SumThreshold step are preserved when antenna zero-amplitude dropouts are flagged
- Output is accepted 'immediately' by AIPS calibration without intermediate manual reformatting

## Limitations

- Zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans are defects inherent to the e-MERLIN system, so the e-MERLIN-specific flagging steps do not generalise to other arrays even though SERPent itself is suitable for any radio interferometric array data
- Flagging throughput is bounded: 110 GB day−1 on a 'high-end' computer (16 CPUs, 100 GB memory), ~6.9 GB CPU−1 day−1

## Evidence

- [abstract] SERPent is written in the Parseltongue language enabling interaction with the Astronomical Image Processing Software (AIPS) program: "SERPent is written in the Parseltongue language enabling interaction with the Astronomical Image Processing Software (AIPS) program"
- [abstract] Output ready for AIPS calibration: "with the output immediately ready for AIPS calibration"
- [readme] SERPent written in ParselTongue and Python interacting with AIPS: "The SERPent software is written in ParselTongue and Python to interact directly with data in the AIPS environment."
- [readme] SERPent suitability scope: "SERPent is suitable for use with any radio interferometric array data."
- [readme] Algorithms utilised by SERPent: "It utilises the algorithms described in the articles "Post-correlation radio frequency interference classification methods'" (Offringa et al. 2010, MNRAS, 405, 155-167)"
