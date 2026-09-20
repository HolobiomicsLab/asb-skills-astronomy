---
name: bbh-mass-distribution-extraction
description: Use when comparing BBH merger-rate yields or mass extremes across COMPAS
  model variants (e.g. Fiducial vs optimistic CE with surviving Hertzsprung-gap donors)
  at low metallicities such as Z=10% or 5% solar.
when_to_use_negative:
- 'When an accurate, observationally comparable merger rate is needed: the paper''s
  rate comparison is approximate and skips convolution of the metallicity-specific
  redshift-dependent star formation rate with the time delay distribution.'
- When the model requires stellar rotation, tides, or wind accretion onto a companion
  — the Fiducial model excludes all of these.
- 'When studying masses above 50 solar masses with SSE-based single-star tracks: the
  single-star model grid extends only to 50 solar masses and is extrapolated beyond
  that.'
edam_operation: http://edamontology.org/operation_3659
tools:
- name: COMPAS
  role: Rapid binary population synthesis code whose Fiducial and optimistic-CE runs
    supply the BBH populations to filter, normalize, and compare
  repo: http://github.com/TeamCOMPAS/COMPAS
- name: SSE
  role: Analytical Hurley et al. single-star evolution fits underlying the stellar
    evolution prescriptions used in the COMPAS simulations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:1423438972274ef0cf17795c52fc251be885f465b824fd61ec8423225aa17241
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - SSE
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bbh-mass-distribution-extraction

## Summary

Extracts and normalizes binary black hole (BBH) merger counts and maximum total masses from COMPAS rapid binary population synthesis runs, enabling quantitative comparison between model variants (e.g. Fiducial pessimistic vs optimistic common envelope) at fixed metallicity.

## When to use

Use when comparing BBH merger-rate yields or mass extremes across COMPAS model variants (e.g. Fiducial vs optimistic CE with surviving Hertzsprung-gap donors) at low metallicities such as Z=10% or 5% solar.

## When NOT to use

- When an accurate, observationally comparable merger rate is needed: the paper's rate comparison is approximate and skips convolution of the metallicity-specific redshift-dependent star formation rate with the time delay distribution.
- When the model requires stellar rotation, tides, or wind accretion onto a companion — the Fiducial model excludes all of these.
- When studying masses above 50 solar masses with SSE-based single-star tracks: the single-star model grid extends only to 50 solar masses and is extrapolated beyond that.

## Inputs

- COMPAS simulation output (Fiducial run) from http://www.sr.bham.ac.uk/compas/
- COMPAS simulation output (optimistic-CE run with surviving Hertzsprung-gap donors) from http://www.sr.bham.ac.uk/compas/
- Metallicity grid values: Z=10% solar, Z=5% solar, and Z=0.002 for maximum-mass checks

## Outputs

- ce_model_comparison.csv with model, metallicity, and value columns containing optimistic/Fiducial rate ratios and maximum total BBH masses
- Total BBH merger counts normalized per 1e9 solar masses of star formation per model per metallicity
- Maximum total BBH mass (solar masses) per model at Z=0.002

## How to apply

Retrieve the published COMPAS simulation outputs (Fiducial and the optimistic-CE run in which Hertzsprung-gap donors survive common envelope) from http://www.sr.bham.ac.uk/compas/. In each run, filter double compact objects to BBHs that merge within a Hubble time, at Z=10% solar and Z=5% solar, and normalize counts per 1e9 solar masses of star formation (Fiducial reference: ~2e4 at 10% solar, ~3e4 at 5% solar). Compute the optimistic/Fiducial total BBH merger-rate ratio at each metallicity, and additionally the maximum total BBH mass at Z=0.002 in each model. Write the ratios and maximum masses (with model, metallicity, and value columns) to a CSV with pandas. Rationale: the optimistic CE treatment predicts total BBH merger rates ~3x higher than Fiducial at Z=10%Z☉ and ~2x higher at Z=5%Z☉, so these ratios are the direct, paper-reported signature that the filtering and normalization were applied consistently across models.

## Related tools

- **COMPAS** (Rapid binary population synthesis code whose Fiducial and optimistic-CE runs supply the BBH populations to filter, normalize, and compare) — http://github.com/TeamCOMPAS/COMPAS
- **SSE** (Analytical Hurley et al. single-star evolution fits underlying the stellar evolution prescriptions used in the COMPAS simulations)

## Examples

```
import pandas as pd; ratios = pd.DataFrame({'model': models, 'metallicity': metallicities, 'value': values}); ratios.to_csv('ce_model_comparison.csv', columns=['model','metallicity','value'])
```

## Evaluation signals

- Optimistic/Fiducial total BBH merger-rate ratio must be ~3 at Z=10% solar and ~2 at Z=5% solar, matching the paper's reported values.
- Fiducial-run counts must reproduce ~2e4 merging BBHs per 1e9 solar masses of star formation at 10% solar metallicity and ~3e4 at 5% solar metallicity.
- Maximum total BBH mass at Z=0.002 must rise from ~50 solar masses (Fiducial) to ~60 solar masses (optimistic CE).
- Counts must be normalized per 1e9 solar masses of star formation before ratios are computed, otherwise model-to-model ratios are invalid.
- ce_model_comparison.csv must contain exactly the model, metallicity, and value columns with one row per (model, metallicity) statistic.

## Limitations

- The merger-rate comparison to LIGO is approximate: an accurate calculation requires convolving the metallicity-specific redshift-dependent star formation rate with the time delay distribution, which was not done.
- Only individual parameters were varied one at a time relative to the Fiducial model; a full exploration of the multidimensional parameter space (e.g. alpha, fLBV) was deferred to future studies.
- The SSE single-star grid extends only to 50 solar masses and is extrapolated above this limit, affecting the heaviest BBH progenitors.
- Stellar rotation, tides, and wind accretion by a companion are not included in the Fiducial model.

## Evidence

- [discussion] Optimistic CE rate enhancement: "the optimistic CE treatment predicts total BBHs merger rates, which are ∼3 times higher than the Fiducial model at Z=10%Z⊙ and ∼2 times higher at Z=5%Z⊙"
- [discussion] Fiducial normalized yield: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [discussion] Rate comparison caveat: "An accurate calculation of the merger rate requires the convolution of the metallicity-specific redshift-dependent star formation rate with the time delay distribution"
- [discussion] Simulation output availability: "We make the results of our simulations available at http://www.sr.bham.ac.uk/compas/."
- [readme] COMPAS code scope: "COMPAS draws properties for a binary star system from a set of initial distributions, and evolves it from zero-age main sequence to the end of its life as two compact remnants."
