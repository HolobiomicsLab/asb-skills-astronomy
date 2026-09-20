---
name: metallicity-dependent-rate-comparison
description: Use when you need BBH merger-rate sensitivity to a single varied physics
  parameter (alpha_lambda, fLBV, CE treatment, Fryer SN model) or metallicity, compared
  against the COMPAS Fiducial baseline per 1e9 solar masses of star formation.
when_to_use_negative:
- The question is formation feasibility or mass-ratio statistics of a population at
  fixed parameters, not rate sensitivity to a varied parameter
- You need an observationally comparable cosmic merger-rate density in Gpc^-3 yr^-1
  — that requires convolving metallicity-specific redshift-dependent star formation
  rate with the time-delay distribution, which this skill explicitly does not do
- You plan to vary multiple model parameters simultaneously; the article's approach
  varies individual parameters independently only
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: COMPAS
  role: Rapid binary population synthesis code used to generate merging-BBH populations
    at Z=0.002 and Z=0.001 under fiducial and single-parameter-varied configurations
  repo: http://github.com/TeamCOMPAS/COMPAS
- name: STROOPWAFEL
  role: COMPAS adaptive importance-sampling algorithm used to efficiently sample the
    rare subpopulation of BBHs that merge within a Hubble time
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/ncomms14906
    title: Formation of the first three gravitational-wave observations through isolated
      binary evolution
  source_content_hash: sha256:4333a79c8cce795810ad9646d3739ad7a8d1111854ec393369d1c6e434a82b89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMPAS
  - STROOPWAFEL
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metallicity-dependent-rate-comparison

## Summary

Quantifies how COMPAS-predicted binary black hole merger rates (total and per-event) change when a single physics parameter such as the common-envelope ejection efficiency alpha_lambda is varied relative to the Fiducial model, at different metallicities (Z=0.002 and Z=0.001, i.e. 10% and 5% solar).

## When to use

Use when you need BBH merger-rate sensitivity to a single varied physics parameter (alpha_lambda, fLBV, CE treatment, Fryer SN model) or metallicity, compared against the COMPAS Fiducial baseline per 1e9 solar masses of star formation.

## When NOT to use

- The question is formation feasibility or mass-ratio statistics of a population at fixed parameters, not rate sensitivity to a varied parameter
- You need an observationally comparable cosmic merger-rate density in Gpc^-3 yr^-1 — that requires convolving metallicity-specific redshift-dependent star formation rate with the time-delay distribution, which this skill explicitly does not do
- You plan to vary multiple model parameters simultaneously; the article's approach varies individual parameters independently only

## Inputs

- COMPAS Fiducial-model DoubleCompactObject outputs at Z=0.002 and Z=0.001 (from the COMPAS Zenodo community deposit or freshly generated runs)
- COMPAS varied-parameter runs (e.g. alpha_lambda=0.01, optimistic CE, fLBV=1e-3 Msun/yr) at the same metallicities
- Component masses of GW150914, GW151226 and LVT151012 to define event-analogue subsamples

## Outputs

- CSV table of total BBH merger rate (mergers per Hubble time per 1e9 Msun star formation) and per-event-analogue counts at each metallicity for fiducial vs varied parameters
- Drop/enhancement factors relative to the Fiducial model (e.g. ~2x at Z=0.002, ~4x at Z=0.001 for alpha_lambda=0.01)
- Maximum merging-BBH mass per run (e.g. shifts from ~50 to ~60 Msun at Z=0.002 and ~75 to ~65 Msun at Z=0.001 under alpha_lambda=0.01)

## How to apply

Run COMPAS at Z=0.002 and Z=0.001 with STROOPWAFEL adaptive importance sampling, first reconstructing the Fiducial model (Kroupa IMF up to 100 Msun, uniform mass-ratio, flat-in-the-log semi-major-axis, zero eccentricity, all stars in binaries), then re-running with exactly one parameter changed (e.g. alpha_lambda=0.01 instead of 0.1). From each run count BBHs merging within a Hubble time per 1e9 Msun of star formation, plus per-event counts for GW150914-, GW151226- and LVT151012-like analogues and the maximum merging-BBH mass. Record drop/enhancement factors in a CSV against the fiducial baseline: expect total rates ~2e4 per 1e9 Msun at 10% solar and ~3e4 at 5% solar; alpha_lambda=0.01 drops the total rate by ~2x at 10% solar and ~4x at 5% solar, with per-event drops of ~50 (GW150914-like), ~25 (GW151226-like) and ~4 (LVT151012-like); an optimistic CE treatment raises totals ~3x and ~2x respectively; raising fLBV from 1.5e-4 to 1e-3 Msun/yr leaves the total rate unchanged but cuts LVT151012-like mergers by ~10x.

## Related tools

- **COMPAS** (Rapid binary population synthesis code used to generate merging-BBH populations at Z=0.002 and Z=0.001 under fiducial and single-parameter-varied configurations) — http://github.com/TeamCOMPAS/COMPAS
- **STROOPWAFEL** (COMPAS adaptive importance-sampling algorithm used to efficiently sample the rare subpopulation of BBHs that merge within a Hubble time)

## Evaluation signals

- Tabulated total-rate drop factor at 10% solar metallicity reproduces the reported factor of ~2 for alpha_lambda=0.01 versus the Fiducial model
- Total-rate drop factor at 5% solar metallicity reproduces the reported factor of ~4
- Fiducial baseline yields ~2e4 merging BBHs per 1e9 Msun at 10% solar metallicity and ~3e4 at 5% solar metallicity
- CSV contains rate and maximum-mass entries for all three event analogues (GW150914-, GW151226-, LVT151012-like) at both metallicities, with per-event drop factors ~50, ~25 and ~4 under alpha_lambda=0.01
- Exactly one parameter differs between each varied run and the Fiducial baseline (single-parameter-variation design)

## Limitations

- An accurate merger-rate comparison requires convolving metallicity-specific redshift-dependent star formation rate with the time-delay distribution, which was not performed; the reported comparison is approximate
- Only individual parameters were varied (alpha_lambda, fLBV, CE treatment, supernova model); full exploration of the multidimensional parameter space is deferred to future studies
- The SSE single-star grid extends only to 50 solar masses and is extrapolated above that limit; stellar rotation, tides and wind accretion by the companion are not included in the Fiducial model

## Evidence

- [discussion] alpha_lambda sensitivity at both metallicities: "changing the value of αλ from the fiducial 0.1 to 0.01. At 10% solar metallicity we find the total BBHs merger rate drops by a factor of ∼2"
- [discussion] fiducial rate normalisation per unit star formation: "We form ∼2 × 104 BBHs that merge in a Hubble time per 1 × 109 solar masses of star formation at 10% solar metallicity"
- [discussion] optimistic common-envelope contrast: "the optimistic CE treatment predicts total BBHs merger rates, which are ∼3 times higher than the Fiducial model at Z=10%Z⊙"
- [discussion] LBV wind parameter effect on per-event rates: "does not significantly change the total BBHs merger rate; nevertheless, the number of BBHs mergers similar to LVT151012 was reduced by a factor of ∼10"
- [discussion] single-parameter-variation methodology: "here we follow the common approach273637 of varying individual parameters independently and assessing their impact relative to the Fiducial model"
- [readme] STROOPWAFEL importance sampling is part of the COMPAS workflow: "If you use COMPAS's importance sampling algorithm STROOPWAFEL, please cite"
