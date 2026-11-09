---
name: metallicity-dependent-population-synthesis-tabulation
description: Use when comparing COMPAS BPS outputs across metallicity or common-envelope
  prescriptions to quantify how many BBHs merge within a Hubble time per 1e9 solar
  masses of star formation.
when_to_use_negative:
- The goal is an observational BBH merger-rate density (Gpc^-3 per year) convolved
  with redshift-dependent star formation history — this skill only tabulates per-metallicity
  counts, which the paper notes is only an approximate rate comparison
- The analysis targets channel-specific sensitivity (e.g. LVT151012-like systems under
  varying LBV mass loss) rather than total merging-BBH counts across CE prescriptions
- A full multi-dimensional parameter-space exploration (e.g. varying alpha and fLBV
  jointly) is needed — the paper only varies individual parameters independently
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to generate the Fiducial and optimistic-CE
    BBH populations at fixed metallicities
  repo: https://github.com/TeamCOMPAS/COMPAS
- name: SSE
  role: Hurley et al. analytical single-star evolution fits used within COMPAS to
    evolve individual stars in the population
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:ead7f4afded366d494c4676ea62c5390d17788d43c688f047fc6b714a7a4e6c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamCOMPAS/COMPAS
  tools:
  - COMPAS
  - SSE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metallicity-dependent-population-synthesis-tabulation

## Summary

Tabulates COMPAS rapid binary population synthesis results for merging binary black holes at fixed metallicities (e.g. 10% and 5% solar), normalized per 1e9 solar masses of star formation, and compares model variants (Fiducial vs optimistic common envelope) via rate ratios.

## When to use

Use when comparing COMPAS BPS outputs across metallicity or common-envelope prescriptions to quantify how many BBHs merge within a Hubble time per 1e9 solar masses of star formation.

## When NOT to use

- The goal is an observational BBH merger-rate density (Gpc^-3 per year) convolved with redshift-dependent star formation history — this skill only tabulates per-metallicity counts, which the paper notes is only an approximate rate comparison
- The analysis targets channel-specific sensitivity (e.g. LVT151012-like systems under varying LBV mass loss) rather than total merging-BBH counts across CE prescriptions
- A full multi-dimensional parameter-space exploration (e.g. varying alpha and fLBV jointly) is needed — the paper only varies individual parameters independently

## Inputs

- COMPAS simulation output (h5/DataFrame) from the Fiducial run at Z=10% solar and Z=5% solar, including BBH merger times and component masses
- COMPAS simulation output from the optimistic-CE run (Hertzsprung-gap donors survive common envelope) at the same metallicities
- Normalization factor of 1e9 solar masses of star formation and a Hubble-time merger cutoff

## Outputs

- ce_model_comparison.csv with columns model, metallicity, total BBH merger count per 1e9 Msun of star formation, optimistic/Fiducial rate ratio, and maximum total BBH mass at Z=0.002

## How to apply

Retrieve the published COMPAS simulation results (Fiducial and optimistic-CE runs) from http://www.sr.bham.ac.uk/compas/. For each run and each metallicity (10% and 5% solar), filter double compact-object outputs for BBHs that merge within a Hubble time and normalize counts per 1e9 solar masses of star formation; the Fiducial references are ~2e4 merging BBHs at 10% solar and ~3e4 at 5% solar. Repeat identically for the optimistic-CE model in which Hertzsprung-gap donors survive common envelope. Compute the optimistic/Fiducial total BBH merger-rate ratio per metallicity, and additionally record the maximum total BBH mass at Z=0.002 for each model. Write the results to ce_model_comparison.csv with model, metallicity, and value columns using pandas. Validate against the paper: ratios must be ~3 at 10% solar and ~2 at 5% solar, and the Z=0.002 maximum total mass must rise from ~50 (Fiducial) to ~60 (optimistic) solar masses.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to generate the Fiducial and optimistic-CE BBH populations at fixed metallicities) — https://github.com/TeamCOMPAS/COMPAS
- **SSE** (Hurley et al. analytical single-star evolution fits used within COMPAS to evolve individual stars in the population)

## Examples

```
import pandas as pd; df[df.t_merge < 13.8e9].groupby(['model','metallicity']).size().to_csv('ce_model_comparison.csv')
```

## Evaluation signals

- Optimistic/Fiducial total BBH merger-rate ratio must be ~3 at Z=10% solar
- Optimistic/Fiducial total BBH merger-rate ratio must be ~2 at Z=5% solar
- Fiducial counts must match ~2e4 merging BBHs per 1e9 Msun of star formation at 10% solar and ~3e4 at 5% solar
- Maximum total BBH mass at Z=0.002 must rise from ~50 (Fiducial) to ~60 (optimistic) solar masses
- ce_model_comparison.csv must contain model, metallicity, and value columns and only BBHs merging within a Hubble time

## Limitations

- The single-star model grid underlying SSE extends only to 50 solar masses and is extrapolated above that limit
- Stellar rotation and tides are not included in the Fiducial model, and wind accretion by a companion is not accounted for
- The merger-rate comparison to LIGO is approximate; an accurate rate requires convolving metallicity-specific redshift-dependent star formation rate with the time delay distribution
- Only individual parameters were varied relative to the Fiducial model; full exploration of the multidimensional parameter space was deferred

## Evidence

- [discussion] We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [discussion] optimistic vs fiducial rate ratios: "the optimistic CE treatment predicts total BBHs merger rates, which are ∼3 times higher than the Fiducial model at Z=10%Z⊙ and ∼2 times higher at Z=5%Z⊙"
- [readme] COMPAS availability: "Simulations in this paper made use of the COMPAS rapid binary population synthesis code (version X.X.X), which is freely available at http://github.com/TeamCOMPAS/COMPAS_."
- [discussion] SSE grid limit: "the original grid of single star models extends only to 50 solar masses. We extrapolate above this limit, as described in Hurley et al.39"
