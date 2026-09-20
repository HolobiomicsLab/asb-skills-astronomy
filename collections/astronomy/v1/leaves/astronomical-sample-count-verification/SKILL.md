---
name: astronomical-sample-count-verification
description: Use when a paper reports inventory-style counts (N spectra, N targets,
  N RVs) for a public data release and the counts must be independently re-derived
  from the release metadata and validated against the published values.
when_to_use_negative:
- The data products are uncalibrated raw frames with no release-level metadata tables
  to enumerate
- The claimed counts refer to planets or occurrence rates (e.g. 1.44 ± 0.20 planets
  per star) rather than data-product inventories, which require MCMC/occurrence analysis,
  not enumeration
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: Python
  role: Loading DR1 metadata tables, enumerating spectra, grouping by target, and
    counting high-level RV products
- name: CARMENES
  role: Instrument whose GTO survey and Data Release 1 provide the spectral and RV
    observations being counted
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:a0ae7eb02434e40b8472ee6464d459c3c5c11eb8ce19aa39ea4afa7df5474cc3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CARMENES
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# astronomical-sample-count-verification

## Summary

Reproduces the published inventory counts of a spectroscopic survey data release by enumerating raw spectra, unique targets, and derived high-level products in Python. Grounded in the CARMENES DR1 release (19,633 spectra / 362 targets; 18,642 RVs / 345 targets).

## When to use

Use when a paper reports inventory-style counts (N spectra, N targets, N RVs) for a public data release and the counts must be independently re-derived from the release metadata and validated against the published values.

## When NOT to use

- The data products are uncalibrated raw frames with no release-level metadata tables to enumerate
- The claimed counts refer to planets or occurrence rates (e.g. 1.44 ± 0.20 planets per star) rather than data-product inventories, which require MCMC/occurrence analysis, not enumeration

## Inputs

- CARMENES Data Release 1 (DR1) metadata tables (raw, pipeline-processed, and high-level products; GTO 2016-2020)
- DR1 radial-velocity time-series product tables (precise RVs per target)

## Outputs

- Summary table with four counts: total spectra, unique spectral targets, total precise RVs, unique RV targets
- Validation report comparing each count against the published DR1 inventory (19,633/362; 18,642/345)

## How to apply

Access the public CARMENES Data Release 1, which makes available all observations from the GTO survey (2016-2020) — raw, pipeline-processed, and high-level products. Load the DR1 metadata tables in Python and enumerate individual spectra collected during the GTO period, then group spectra by stellar target to count unique targets. Separately load the high-level radial-velocity time series and count precise RVs and the unique targets for which RVs were derived. Assemble the four counts into a summary table and validate each against the published inventory: 19,633 spectra for 362 targets and 18,642 precise RVs for 345 targets. The rationale is that a data-release inventory is a self-describing invariant — every count in the abstract must be recoverable from the release's own tables.

## Related tools

- **Python** (Loading DR1 metadata tables, enumerating spectra, grouping by target, and counting high-level RV products)
- **CARMENES** (Instrument whose GTO survey and Data Release 1 provide the spectral and RV observations being counted)

## Examples

```
counts = {'spectra': len(dr1_spectra), 'targets': dr1_spectra['target'].nunique(), 'rvs': len(dr1_rvs), 'rv_targets': dr1_rvs['target'].nunique()}; assert counts == {'spectra': 19633, 'targets': 362, 'rvs': 18642, 'rv_targets': 345}
```

## Evaluation signals

- Spectra count equals exactly 19,633 after enumerating GTO-period (2016-2020) observations in the DR1 tables
- Unique spectral targets after grouping by star equals 362
- Precise RV count equals 18,642 in the high-level data products
- Unique targets with derived RVs equals 345
- All four counts in the summary table match the published DR1 inventory with no residual mismatch

## Limitations

- Counts are specific to the DR1 GTO sample (2016-2020); later releases or different survey samples will not reproduce these values
- Distinct spectrum-target counts vs. RV-target counts (362 vs 345) must be treated separately — some targets have spectra but no derived precise RVs

## Evidence

- [abstract] DR1 inventory counts (abstract): "spanning from 2016 to 2020, during which 19 633 spectra for a sample of 362 targets were collected"
- [abstract] RV high-level products (abstract): "The data were pipeline-processed, and high-level data products, including 18 642 precise RVs for 345 targets, were derived."
- [abstract] DR1 scope definition (abstract): "We present the CARMENES Data Release 1 (DR1), which makes public all observations obtained during the GTO of the CARMENES survey."
- [abstract] Survey target completeness (abstract): "about 70% of all known M dwarfs within 10 pc and accessible from Calar Alto were included"
