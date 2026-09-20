---
name: parseltongue-aips-scripting
description: Use when automating reduction or flagging of radio interferometric visibility
  data (e.g. UVFITS files from e-MERLIN L-band 1.3–1.8 GHz or C-band 4–8 GHz) that
  must be read, modified, and written back within AIPS without manual interaction.
when_to_use_negative:
- The data are not radio interferometric visibility data accessible from AIPS — the
  README states SERPent's ParselTongue/Python approach exists 'to interact directly
  with data in the AIPS environment'.
- A fully manual, interactive AIPS visual flagging session is preferred over automated
  scripted processing.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: SERPent
  role: Automated reduction and RFI-mitigation pipeline whose ParselTongue scripting
    this skill describes
  repo: https://github.com/daniellefenech/SERPent
- name: Parseltongue
  role: Scripting language used to read and write UVFITS data through the AIPS environment
- name: AIPS
  role: Astronomical Image Processing Software environment in which visibility data
    are manipulated
- name: Python
  role: Language of the 'out of the box' SERPent script layered on ParselTongue
- name: e-MERLIN
  role: Interferometric array whose commissioning L-band and C-band visibility datasets
    are processed
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.09.001
    title: 'SERPent: Automated reduction and RFI-mitigation software for e-MERLIN'
  source_content_hash: sha256:44c697dd7acaf508a52b7c2dfb6f0d7f67a59be16da349c3a9d7d6d12214d4a8
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# parseltongue-aips-scripting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scripting methodology for programmatically manipulating interferometric visibility data inside the AIPS environment using ParselTongue and Python, as operationalised by the SERPent pipeline for e-MERLIN datasets.

## When to use

Use when automating reduction or flagging of radio interferometric visibility data (e.g. UVFITS files from e-MERLIN L-band 1.3–1.8 GHz or C-band 4–8 GHz) that must be read, modified, and written back within AIPS without manual interaction.

## When NOT to use

- The data are not radio interferometric visibility data accessible from AIPS — the README states SERPent's ParselTongue/Python approach exists 'to interact directly with data in the AIPS environment'.
- A fully manual, interactive AIPS visual flagging session is preferred over automated scripted processing.

## Inputs

- UVFITS visibility file from an e-MERLIN commissioning observation (L-band 1.3–1.8 GHz or C-band 4–8 GHz)
- AIPS visibility/flag tables accessed via ParselTongue

## Outputs

- Flagged visibility dataset exported as a UVFITS file (e.g. flagged_visibilities.uvf) ready for AIPS calibration
- Flag classifications for RFI-affected samples, antenna zero-amplitude dropouts, and Lovell phase-calibrator stationary scans

## How to apply

Write ParselTongue/Python scripts that load UVFITS visibility files into the AIPS data area, then operate on the visibility records programmatically rather than through the AIPS interactive interface. For SERPent-style RFI mitigation, reorder visibility amplitudes into time-by-frequency sequences per baseline before applying the SumThreshold detection of Offringa et al. 2010 (MNRAS, 405, 155-167) and the morphological algorithm of Offringa et al. 2012 (A&A, 539, A95); propagate detected flags back to the AIPS visibility records, including antenna zero-amplitude dropouts and Lovell telescope phase calibrator stationary scans. Finally export the flagged dataset back to UVFITS (e.g. flagged_visibilities.uvf) so it is immediately ready for AIPS calibration. The rationale is that ParselTongue provides a scriptable bridge (SERPent is 'written in the Parseltongue language enabling interaction with the Astronomical Image Processing Software (AIPS) program'), keeping the workflow free of compilers and reproducible as a simple 'out of the box' Python script.

## Related tools

- **SERPent** (Automated reduction and RFI-mitigation pipeline whose ParselTongue scripting this skill describes) — https://github.com/daniellefenech/SERPent
- **Parseltongue** (Scripting language used to read and write UVFITS data through the AIPS environment)
- **AIPS** (Astronomical Image Processing Software environment in which visibility data are manipulated)
- **Python** (Language of the 'out of the box' SERPent script layered on ParselTongue)
- **e-MERLIN** (Interferometric array whose commissioning L-band and C-band visibility datasets are processed)

## Evaluation signals

- The exported output file exists and is a valid UVFITS file loadable into AIPS calibration.
- Flagged data fraction is consistent with reported RFI levels: <20%–25% for problematic L-band observations and <5% for the quieter C-band.
- All original RFI, dropout, and stationary-scan flags are preserved (no flags lost) between pipeline steps.
- The script runs without compilers as a plain Python/ParselTongue script in the AIPS environment.
- Throughput is in line with the reported flagging rate of ~110 GB day^-1 (~6.9 GB CPU^-1 day^-1 on a 16-CPU, 100 GB memory machine).

## Limitations

- The reported flagging rate is 110 GB day^-1 on a 'high-end' computer (16 CPUs, 100 GB memory); larger volumes may need comparable resources or take proportionally longer.
- The flagged output must remain immediately ready for AIPS calibration — intermediate custom formats outside AIPS break the workflow contract.
- Discussion-section content (e.g. extension to SKA pathfinders MeerKAT and ASKAP) is not covered by the available abstract/README material.

## Evidence
