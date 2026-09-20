---
name: radial-velocity-inventory-tabulation
description: Use when reproducing or auditing a published spectrograph data release
  inventory, e.g. you have FITS spectra or RV time series from a survey like CARMENES
  DR1 and must report per-target counts of spectra and RVs.
when_to_use_negative:
- The analysis goal is characterizing the RV measurements themselves (e.g. planet
  detection, CCF profiling with raccoonccf) rather than enumerating release inventory
  counts
- The data are proprietary or pre-release, so no published inventory exists to validate
  counts against
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Python
  role: Load DR1 metadata tables, enumerate spectra, group by stellar target, and
    count unique targets and precise RVs
- name: CARMENES
  role: Source spectrograph whose DR1 release provides the raw, pipeline-processed,
    and high-level products being inventoried
- name: raccoon
  role: Open source Python package implementing the CCF method used to derive the
    RV time series inventoried by this skill
  repo: github.com/mlafarga/raccoon
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:3efb3c4b774a1aaab9c33e2741cba9e36b63fe52d1f024ef43087232f4de158d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CARMENES
  - Python
  - raccoon
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# radial-velocity-inventory-tabulation

## Summary

Enumerate a spectroscopic survey's data-product inventory by counting raw/pipeline spectra and derived radial-velocity measurements grouped by unique stellar target, and validate each count against the published release numbers (for CARMENES GTO DR1: 19,633 spectra / 362 targets; 18,642 precise RVs / 345 targets).

## When to use

Use when reproducing or auditing a published spectrograph data release inventory, e.g. you have FITS spectra or RV time series from a survey like CARMENES DR1 and must report per-target counts of spectra and RVs.

## When NOT to use

- The analysis goal is characterizing the RV measurements themselves (e.g. planet detection, CCF profiling with raccoonccf) rather than enumerating release inventory counts
- The data are proprietary or pre-release, so no published inventory exists to validate counts against

## Inputs

- CARMENES Data Release 1 (DR1) metadata tables of GTO observations (2016-2020)
- Reduced FITS spectra (e.g. CARMENES VIS/NIR pipeline-processed products)
- High-level data products: radial-velocity time series per target

## Outputs

- Count of individual spectra collected during the survey period (e.g. 19,633)
- Count of unique stellar targets observed (e.g. 362)
- Count of precise RVs derived (e.g. 18,642)
- Count of targets with derived RVs (e.g. 345)
- Summary table comparing each tabulated count with the published inventory

## How to apply

Access the data release (e.g. CARMENES DR1, which makes public all GTO observations from 2016-2020 including raw, pipeline-processed, and high-level products). Load the release metadata tables in Python and enumerate the individual spectra collected during the survey period. Group the spectra by stellar target to count unique targets observed. Then load the high-level data products — the radial-velocity time series — and count the precise RVs and the unique targets for which RVs were derived (18,642 RVs for 345 targets in DR1). Assemble the four counts into a summary table and compare each value against the published inventory as validation.

## Related tools

- **Python** (Load DR1 metadata tables, enumerate spectra, group by stellar target, and count unique targets and precise RVs)
- **CARMENES** (Source spectrograph whose DR1 release provides the raw, pipeline-processed, and high-level products being inventoried)
- **raccoon** (Open source Python package implementing the CCF method used to derive the RV time series inventoried by this skill) — github.com/mlafarga/raccoon

## Examples

```
raccoonccf PATH/TO/OBS/car*vis_A.fits CARM_VIS J07274+052default --obj OBJ --filtell default --rvshift header --fcorrorders obshighsnr --plot_sv --verbose
```

## Evaluation signals

- Tabulated spectra count equals the published release value (19,633 for CARMENES DR1 GTO)
- Tabulated unique-target count from grouping spectra equals 362
- Count of precise RVs from the high-level products equals 18,642
- Count of targets with derived RVs equals 345
- All four counts must match the reported DR1 inventory simultaneously; any mismatch indicates a loading/grouping error

## Limitations

- Counts depend on correctly distinguishing raw/pipeline spectra from high-level RV products; conflating them breaks the validation against the published inventory
- Targets observed but lacking precise RVs (362 vs 345 in DR1) must be kept as separate counts
- Survey period boundaries (2016-2020 GTO) must be respected when filtering observations

## Evidence

- [abstract] 19,633 spectra for 362 targets; 18,642 RVs derived: "spanning from 2016 to 2020, during which 19 633 spectra for a sample of 362 targets were collected"
- [abstract] Data pipeline-processed and RVs derived: "The data were pipeline-processed, and high-level data products, including 18 642 precise RVs for 345 targets, were derived."
- [abstract] DR1 makes public all GTO observations: "We present the CARMENES Data Release 1 (DR1), which makes public all observations obtained during the GTO of the CARMENES survey."
- [readme] raccoon derives RVs from CCF: "derive radial velocities (RVs) and activity indicators from the CCF"
