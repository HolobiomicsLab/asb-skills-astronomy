---
name: survey-target-catalog-summarization
description: 'Use when a public survey data release (e.g. CARMENES DR1) must be inventoried:
  counting raw/pipeline-processed spectra, unique targets, or derived time-series
  products to reproduce or audit the release''s headline numbers.'
when_to_use_negative:
- The analysis goal is deriving new RVs or activity indicators from raw spectra via
  cross-correlation — use the raccoon CCF workflow instead, not inventory counting
- The data release has no published reference inventory to validate against, so counts
  cannot be verified against headline numbers
- You need per-observation scientific quality assessment rather than release-level
  catalog enumeration
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: Python
  role: Load the DR1 metadata tables and high-level products, enumerate and group
    spectra by target, and assemble the count summary table
- name: CARMENES
  role: Spectrograph whose GTO survey observations (2016–2020) constitute the DR1
    release being inventoried
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:6772c12aed2f9dd4fb93408d3313c77cea006e7360e61e9f3a7843da99c6e80a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CARMENES
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# survey-target-catalog-summarization

## Summary

Enumerate the inventory of a spectroscopic survey's public data release by loading its metadata tables, counting spectra and unique stellar targets, and counting derived high-level products (e.g. precise radial-velocity time series) per target, then validating each count against the published release numbers.

## When to use

Use when a public survey data release (e.g. CARMENES DR1) must be inventoried: counting raw/pipeline-processed spectra, unique targets, or derived time-series products to reproduce or audit the release's headline numbers.

## When NOT to use

- The analysis goal is deriving new RVs or activity indicators from raw spectra via cross-correlation — use the raccoon CCF workflow instead, not inventory counting
- The data release has no published reference inventory to validate against, so counts cannot be verified against headline numbers
- You need per-observation scientific quality assessment rather than release-level catalog enumeration

## Inputs

- CARMENES DR1 metadata tables listing all GTO observations (2016–2020), including raw and pipeline-processed spectra
- Reduced CARMENES FITS spectra (e.g. CARM_VIS/CARM_NIR files such as *_vis_A.fits) with target identifiers
- High-level data products: radial-velocity (RV) time series per target

## Outputs

- Summary table with four counts: total spectra (19,633), unique spectral targets (362), precise RVs (18,642), unique RV targets (345)
- Per-target grouping of spectra and RV measurements

## How to apply

Access the public data release — for CARMENES DR1 this is all observations from the guaranteed time observations (GTO) survey spanning 2016–2020, including raw, pipeline-processed (caracal-processed CARM_VIS/CARM_NIR FITS), and high-level products. Load the DR1 metadata tables in Python and enumerate the individual spectra collected during the GTO period, then group spectra by stellar target (CARMENES identifiers like J07274+052) to count unique targets. Separately load the high-level data products — the radial-velocity time series — and count precise RVs and the unique targets for which RVs were derived. Assemble the four counts (spectra, spectral targets, RVs, RV targets) into a summary table and compare each value against the published DR1 inventory: 19 633 spectra for 362 targets and 18 642 precise RVs for 345 targets. The rationale is that the release's abstract-level statistics serve as the ground-truth invariant; any mismatch signals incomplete file enumeration, wrong epoch filtering, or mis-grouping by target.

## Related tools

- **Python** (Load the DR1 metadata tables and high-level products, enumerate and group spectra by target, and assemble the count summary table)
- **CARMENES** (Spectrograph whose GTO survey observations (2016–2020) constitute the DR1 release being inventoried)

## Examples

```
python -c "import pandas as pd; obs = pd.read_csv('dr1_observations.csv'); obs = obs[(obs.date >= '2016-01-01') & (obs.date <= '2020-12-31')]; print(len(obs), obs.target.nunique()); rvs = pd.read_csv('dr1_rvs.csv'); print(len(rvs), rvs.target.nunique())"
```

## Evaluation signals

- Spectra count equals 19,633 and unique spectral targets equal 362, matching the published DR1 inventory
- Precise RV count equals 18,642 and unique RV targets equal 345, matching the published DR1 inventory
- Unique-target grouping is consistent: number of distinct target identifiers in the spectral table equals 362 and in the RV product table equals 345
- All enumerated observations fall within the GTO epoch 2016–2020; counts change if the time filter is wrong
- The RV target count (345) is strictly less than the spectral target count (362), as expected since some targets lack derived precise RVs

## Limitations

- Validation is only possible when the release publishes its inventory counts; without the reference numbers (19,633/362 and 18,642/345) there is no external invariant
- Counts depend on correctly scoping to the GTO period (2016–2020); non-GTO or post-release observations must be excluded

## Evidence

- [abstract] DR1 release scope and inventory counts: "We present the CARMENES Data Release 1 (DR1), which makes public all observations obtained during the GTO of the CARMENES survey."
- [abstract] GTO inventory headline numbers: "spanning from 2016 to 2020, during which 19 633 spectra for a sample of 362 targets were collected"
- [abstract] High-level product counts: "The data were pipeline-processed, and high-level data products, including 18 642 precise RVs for 345 targets, were derived."
- [abstract] Instrument identification: "The CARMENES instrument, installed at the 3.5 m telescope of the Calar Alto Observatory in Almería, Spain, was conceived to deliver high-accuracy radial velocity (RV) measurements"
