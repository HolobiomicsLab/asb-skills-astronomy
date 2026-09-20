---
name: harps-spectral-data-ingestion
description: Use when starting a template-matching RV analysis on ESO HARPS archival
  S2D FITS observations (e.g. of Barnard's star) and a quality-controlled frame set
  must be loaded into SBART before template building.
when_to_use_negative:
- Input spectra are from ESPRESSO or another instrument not covered by the HARPS S2D
  ingestion path (though SBART also supports ESPRESSO data)
- Frames are already reduced, validated, and stored in SBART's internal format, so
  re-running ingestion would duplicate work
- Only 1D merged spectra (e.g. S1D) are available, since the per-order template-matching
  workflow requires S2D order-by-order data
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SBART
  role: Python package (pip install SBART) whose DataClass ingests and quality-filters
    HARPS S2D frames
  repo: https://github.com/iastro-pt/sBART
- name: HARPS
  role: Instrument whose archival S2D observations are the ingested data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:ae18004275f436e172714c57e6496d026cdad608db2a427c98bb3aece280b9b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - HARPS
  - HARPS-TERRA
  - S-BART
  - SBART
  - SERVAL
  - Telfit
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# harps-spectral-data-ingestion

## Summary

Ingests HARPS archival S2D spectral frames into the SBART DataClass while applying frame-quality filtering (order-wise SNR check, warning keyword flags, MAD-based flux outlier flagging), producing the validated input set for stellar-template construction and RV extraction.

## When to use

Use when starting a template-matching RV analysis on ESO HARPS archival S2D FITS observations (e.g. of Barnard's star) and a quality-controlled frame set must be loaded into SBART before template building.

## When NOT to use

- Input spectra are from ESPRESSO or another instrument not covered by the HARPS S2D ingestion path (though SBART also supports ESPRESSO data)
- Frames are already reduced, validated, and stored in SBART's internal format, so re-running ingestion would duplicate work
- Only 1D merged spectra (e.g. S1D) are available, since the per-order template-matching workflow requires S2D order-by-order data

## Inputs

- HARPS archival S2D FITS spectral frames (per-order spectra with header keywords for SNR and warning flags)
- Frame header keyword values (DRS RV estimates, warning KW flags, SNR per order)

## Outputs

- SBART DataClass object holding the quality-filtered HARPS S2D frame set
- Data rejection diagnostics (DataRejectionSummary.png) listing rejected frames and rejection criteria

## How to apply

Load the HARPS archival S2D observations into the SBART DataClass, which parses the FITS frames and their header keywords. Configure frame selection with the order-wise SNR check, reject frames carrying warning keyword flags (e.g. DRS warning keywords), and enable MAD-based flagging of flux outliers so that anomalous exposures are excluded before any RV computation. Optionally apply greater control over which frames are available and sigma-clip RVs based on the DRS estimate to remove outliers relative to the pipeline values. The rationale is that template construction and posterior RV quality depend on the input frame set: pathological orders, telluric-contaminated exposures, or frames flagged by DRS warnings would bias the stellar template and inflate RV scatter. Inspect the DataRejectionSummary diagnostics to confirm which frames were rejected and why before proceeding to template building.

## Related tools

- **SBART** (Python package (pip install SBART) whose DataClass ingests and quality-filters HARPS S2D frames) — https://github.com/iastro-pt/sBART
- **HARPS** (Instrument whose archival S2D observations are the ingested data)

## Examples

```
pip install SBART
```

## Evaluation signals

- The SBART DataClass load completes without errors and the frame set contains the expected number of HARPS S2D frames
- DataRejectionSummary.png exists and every rejected frame is attributable to one of the configured criteria (order-wise SNR, warning KW flags, MAD flux outliers)
- No frame with an active DRS warning keyword or a MAD-flagged flux outlier survives into the template-building stage
- The surviving frame set matches expectations when sigma-clipping against the DRS RV estimates (no outliers relative to DRS values)

## Limitations

- Frame rejection relies on DRS-produced header keywords and SNR values; frames with missing or corrupted keywords cannot be filtered reliably
- MAD-based flux-outlier flagging and NORMALIZATION_MODE choices affect which frames survive, so ingestion settings must be documented for reproducibility
- Telfit/pysynphot installation issues downstream can fail for reference frames for which GDAS profiles cannot be downloaded (relevant when telluric templates are built from the same ingested frames)

## Evidence

- [methods] Load HARPS S2D frames into DataClass with order-wise SNR check: "Added a new condition to check order-wise SNR"
- [methods] Reject frames based on DRS warning keyword flags: "Allow to reject Frames based on warning KW flags that might exist"
- [methods] Flag flux outliers with MAD: "Allow to use MAD to flag flux outliers."
- [readme] SBART is pip-installable: "SBART can be pip-installed:

    pip install SBART"
- [abstract] Validate on HARPS archival observations of Barnard's star: [evidence span withheld — anchor status: not_found]
