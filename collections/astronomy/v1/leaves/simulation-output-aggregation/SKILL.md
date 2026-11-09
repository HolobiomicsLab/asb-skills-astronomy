---
name: simulation-output-aggregation
description: Use when you have COMPAS (or similar BPS) simulation output for multiple
  model variants and metallicities and need per-model BBH merger counts, normalized
  rates, and model-comparison tables.
when_to_use_negative:
- 'When an accurate detectable merger rate is needed: the article notes this requires
  convolution of the metallicity-specific redshift-dependent star formation rate with
  the time delay distribution, which simple count aggregation does not do'
- When the underlying COMPAS runs differ in more than the parameter of interest —
  individual-parameter variations are only meaningful relative to the Fiducial model
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: COMPAS
  role: Rapid binary population synthesis code whose per-model output catalogs (Fiducial,
    optimistic CE) are filtered and aggregated
  repo: http://github.com/TeamCOMPAS/COMPAS_
- name: SSE
  role: Single-star evolution fits (Hurley et al.) underlying the COMPAS stellar evolution
    used in the aggregated runs
- name: STROOPWAFEL
  role: Import sampling algorithm available within COMPAS for the simulations whose
    outputs are aggregated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:81d04edf083a5f2ba1e83d69f7793da1342bca76116d6b1d4364b929e3cb84e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - SSE
  - STROOPWAFEL
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# simulation-output-aggregation

## Summary

Aggregate COMPAS binary population synthesis outputs by filtering merged BBHs within a Hubble time at fixed metallicities, normalizing per 1e9 solar masses of star formation, and comparing model variants (e.g. Fiducial vs optimistic CE) as rate ratios and extrema.

## When to use

Use when you have COMPAS (or similar BPS) simulation output for multiple model variants and metallicities and need per-model BBH merger counts, normalized rates, and model-comparison tables.

## When NOT to use

- When an accurate detectable merger rate is needed: the article notes this requires convolution of the metallicity-specific redshift-dependent star formation rate with the time delay distribution, which simple count aggregation does not do
- When the underlying COMPAS runs differ in more than the parameter of interest — individual-parameter variations are only meaningful relative to the Fiducial model

## Inputs

- COMPAS simulation output files for the Fiducial run (BBH properties: masses, merger times, metallicities)
- COMPAS simulation output files for the optimistic-CE run (HG donors survive common envelope)
- Metallicity labels (Z=10% solar, Z=5% solar, Z=0.002) and star-formation normalization of 1e9 solar masses

## Outputs

- ce_model_comparison.csv with model, metallicity, and value columns (rate ratios and maximum total BBH masses)
- Optimistic/Fiducial total BBH merger-rate ratios per metallicity
- Normalized merging-BBH counts per 1e9 solar masses of star formation per model/metallicity

## How to apply

Retrieve the published COMPAS run outputs (Fiducial and optimistic-CE, in which Hertzsprung-gap donors survive common envelope) from www.sr.bham.ac.uk/compas/ or the COMPAS Zenodo community. For each model and metallicity (Z=10% and Z=5% solar), filter for BBHs that merge within a Hubble time and normalize counts per 1e9 solar masses of star formation (Fiducial reference: ~2e4 at 10% solar, ~3e4 at 5% solar). Compute the optimistic/Fiducial total BBH merger-rate ratio at each metallicity and the maximum total BBH mass (e.g. at Z=0.002, where it rises from ~50 Msun Fiducial to ~60 Msun optimistic). Write model, metallicity, and value columns to a CSV such as ce_model_comparison.csv with pandas, validating the ratios against the paper's reported ~3x and ~2x factors.

## Related tools

- **COMPAS** (Rapid binary population synthesis code whose per-model output catalogs (Fiducial, optimistic CE) are filtered and aggregated) — http://github.com/TeamCOMPAS/COMPAS_
- **SSE** (Single-star evolution fits (Hurley et al.) underlying the COMPAS stellar evolution used in the aggregated runs)
- **STROOPWAFEL** (Import sampling algorithm available within COMPAS for the simulations whose outputs are aggregated)

## Examples

```
import pandas as pd; df = pd.DataFrame({'model':['Fiducial','Optimistic']*2,'metallicity':['Z=0.1Zsun','Z=0.1Zsun','Z=0.05Zsun','Z=0.05Zsun'],'value':[2e4,6e4,3e4,6e4]}); df.to_csv('ce_model_comparison.csv', index=False)
```

## Evaluation signals

- Optimistic/Fiducial total BBH merger-rate ratio is ~3 at Z=10% solar and ~2 at Z=5% solar
- Fiducial normalized counts match ~2e4 merging BBHs per 1e9 solar masses of star formation at 10% solar and ~3e4 at 5% solar
- Maximum total BBH mass at Z=0.002 rises from ~50 (Fiducial) to ~60 (optimistic) solar masses
- Output CSV contains model, metallicity, and value columns with one row per model-metallicity pair
- Each model variant differs from Fiducial by exactly one parameter (e.g. CE treatment), matching the paper's one-at-a-time variation approach

## Limitations

- The single-star model grid underlying SSE extends only to 50 solar masses and is extrapolated above that limit
- Stellar rotation and tides are not included in the Fiducial model
- Wind accretion by a companion is not accounted for
- Merger-rate comparison to LIGO is approximate; full SFR/redshift convolution was deferred

## Evidence

- [discussion] optimistic ratio finding: "the optimistic CE treatment predicts total BBHs merger rates, which are ∼3 times higher than the Fiducial model at Z=10%Z⊙ and ∼2 times higher at Z=5%Z⊙"
- [discussion] normalization reference counts: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [discussion] one-parameter-at-a-time variation rationale: "here we follow the common approach273637 of varying individual parameters independently and assessing their impact relative to the Fiducial model."
- [discussion] simulation results availability: "We make the results of our simulations available at http://www.sr.bham.ac.uk/compas/."
- [readme] data sharing expectation: "make your input settings and output data publicly available by uploading it to the [COMPAS Zenodo community](https://zenodo.org/communities/compas/)"
