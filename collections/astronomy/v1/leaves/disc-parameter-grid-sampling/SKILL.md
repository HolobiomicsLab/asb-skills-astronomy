---
name: disc-parameter-grid-sampling
description: Use when model–data agreement for 12CO fluxes (e.g. against Lupus or
  Upper Sco ALMA samples) is inconclusive and you need to quantify how uncertainties
  in initial disc mass and radius propagate into predicted flux spread.
when_to_use_negative:
- You need physically self-consistent thermochemical disc structures for individual
  discs — use the DALI benchmarking route instead, since the semi-analytical model
  only agrees with DALI within a factor of three.
- The goal is to constrain the underlying disc evolution mechanism (viscosity vs MHD
  winds); the paper notes available data have not given conclusive answers, and initial-condition
  uncertainties limit interpretation.
- The disc sample is outside Lupus/Upper Sco or the initial conditions fall outside
  the ranges parameterized in COpops.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: COpops
  role: Supplies the semi-analytical 12CO flux model and the parameterized initial
    disc mass and radius grids used to build the sensitivity grid
  repo: https://github.com/fzagaria/COpops
- name: ALMA
  role: Source of the observed 12CO fluxes for the Lupus and Upper Sco disc samples
    against which grid-point predictions are compared
- name: DALI
  role: 'Thermochemical benchmark reference: the semi-analytical model is validated
    to agree with DALI within a factor of three'
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:3d496b3e48bc3b06ee8283bbb3fa5e5eda5a1adbbedc49a62f33fb0b39085a3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fzagaria/COpops
  tools:
  - ALMA
  - COpops
  - DALI
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# disc-parameter-grid-sampling

## Summary

Sensitivity analysis for semi-analytical 12CO disc flux predictions by sampling a grid of initial disc mass and radius combinations from the COpops package at Lupus/Upper Sco disc ages, quantifying how initial-condition uncertainties spread predicted fluxes against observed ALMA fluxes.

## When to use

Use when model–data agreement for 12CO fluxes (e.g. against Lupus or Upper Sco ALMA samples) is inconclusive and you need to quantify how uncertainties in initial disc mass and radius propagate into predicted flux spread.

## When NOT to use

- You need physically self-consistent thermochemical disc structures for individual discs — use the DALI benchmarking route instead, since the semi-analytical model only agrees with DALI within a factor of three.
- The goal is to constrain the underlying disc evolution mechanism (viscosity vs MHD winds); the paper notes available data have not given conclusive answers, and initial-condition uncertainties limit interpretation.
- The disc sample is outside Lupus/Upper Sco or the initial conditions fall outside the ranges parameterized in COpops.

## Inputs

- Initial disc mass and characteristic-radius ranges parameterized in the COpops package
- Disc ages for the Lupus and Upper Sco regions
- Observed ALMA 12CO fluxes of the Lupus and Upper Sco disc samples

## Outputs

- initial_condition_sensitivity.csv with per-grid-point 12CO fluxes and spread statistics (factor-of range, percentile spread) of model–data agreement

## How to apply

Load the initial disc mass and characteristic-radius ranges parameterized in the COpops package (github.com/fzagaria/COpops), which models discs with power-law surface-density and temperature profiles. Construct a grid of initial-condition combinations spanning those mass–radius ranges at the relevant Lupus and Upper Sco disc ages. For each grid point, evaluate the semi-analytical 12CO flux model under the optically thick assumption (fluxes scale with disc surface area) instead of running expensive DALI thermochemical models. Compare each grid-point prediction against the observed ALMA 12CO fluxes of the Lupus and Upper Sco samples, then compute spread statistics such as the factor-of range and percentile spread of predicted fluxes and of the model–data agreement. Write per-grid-point fluxes and the spread summary to initial_condition_sensitivity.csv. This addresses the paper's finding that uncertainties on initial conditions limited the interpretation of the observations.

## Related tools

- **COpops** (Supplies the semi-analytical 12CO flux model and the parameterized initial disc mass and radius grids used to build the sensitivity grid) — https://github.com/fzagaria/COpops
- **ALMA** (Source of the observed 12CO fluxes for the Lupus and Upper Sco disc samples against which grid-point predictions are compared)
- **DALI** (Thermochemical benchmark reference: the semi-analytical model is validated to agree with DALI within a factor of three)

## Evaluation signals

- The sensitivity table covers the full parameterized initial-condition grid (all mass–radius combinations at both disc ages).
- Recovered model–data agreement is reported against the benchmark that the semi-analytical model agrees with DALI thermochemical models within a factor of three.
- Spread statistics (factor-of range and percentile spread) of predicted fluxes and of model–data agreement are computed across the grid, not just point estimates.
- Per-grid-point fluxes and the spread summary are persisted to initial_condition_sensitivity.csv.
- Grid predictions remain consistent with the optically thick scaling of 12CO flux with disc surface area.

## Limitations

- The semi-analytical model recovers agreement with DALI only within a factor of three, so flux inferences are approximate rather than thermochemically exact.
- The paper's own conclusion is that uncertainties on the initial conditions limited interpretation of the Lupus and Upper Sco observations; the sensitivity analysis quantifies but does not remove this degeneracy.
- Models and data agree well only if CO depletion is considered; ignoring CO depletion will bias comparisons.
- The available ALMA data have not given conclusive answers on the underlying disc evolution mechanisms (viscosity vs MHD winds).

## Evidence

- [abstract] Semi-analytical model agrees with DALI within factor of three: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [abstract] Initial conditions limited interpretation: "the uncertainties on the initial conditions limited our interpretation of the observations"
- [abstract] Fluxes scale with disc surface area under optically thick assumption: "assuming that 12CO is optically thick, fluxes scale with the disc surface area"
- [readme] Fast flexible alternative to expensive thermochemical models: "running computationally-expensive thermochemical models for hundreds of discs can be prohibitively time-consuming. Here we introduce an alternative method based on modelling semi-analytically the"
- [abstract] CO depletion required for model-data agreement: "Models and data agree well only if CO depletion is considered"
