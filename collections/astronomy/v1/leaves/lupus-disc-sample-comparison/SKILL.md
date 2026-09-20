---
name: lupus-disc-sample-comparison
description: Use when testing disc evolution models against a Lupus-style ALMA survey
  catalog of 12CO fluxes and continuum sizes, or when deciding whether flux- or size-based
  selection yields a larger detectable disc sample.
when_to_use_negative:
- When the goal is to discriminate between viscosity and MHD winds definitively —
  the article reports available ALMA data have not given conclusive answers on the
  underlying disc evolution mechanism
- When the input sample has no ALMA 12CO fluxes or 887-micron disc sizes, since the
  flux-vs-size sample-count comparison requires both criteria
- When computationally-expensive full thermochemical modelling of hundreds of individual
  discs is required — the semi-analytical method trades accuracy for speed (agreement
  only within a factor of three of DALI)
edam_operation: http://edamontology.org/operation_2409
tools:
- name: ALMA
  role: Instrument providing the observed Lupus (and Upper Sco) 12CO line fluxes and
    887-micron disc sizes against which the model tracks are compared
- name: DALI
  role: Thermochemical model used to benchmark the semi-analytical 12CO flux computation
    (agreement within a factor of three)
- name: COpops
  role: Code implementing the semi-analytical CO flux models used to synthesise disc
    populations for the Lupus comparison
  repo: github.com/fzagaria/COpops
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:b18b7ab2bd9f40bfeb5e78ffd6a85d9565e4d3f2ac362a0f30590d0ac1a1a3b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA
  - COpops
  - DALI
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# lupus-disc-sample-comparison

## Summary

Compares semi-analytical 12CO flux model tracks (viscous and MHD-wind, with and without CO depletion) against the ALMA Lupus disc sample of 12CO line fluxes and 887-micron disc sizes, and quantifies how the usable sample grows when fluxes replace sizes as the observable.

## When to use

Use when testing disc evolution models against a Lupus-style ALMA survey catalog of 12CO fluxes and continuum sizes, or when deciding whether flux- or size-based selection yields a larger detectable disc sample.

## When NOT to use

- When the goal is to discriminate between viscosity and MHD winds definitively — the article reports available ALMA data have not given conclusive answers on the underlying disc evolution mechanism
- When the input sample has no ALMA 12CO fluxes or 887-micron disc sizes, since the flux-vs-size sample-count comparison requires both criteria
- When computationally-expensive full thermochemical modelling of hundreds of individual discs is required — the semi-analytical method trades accuracy for speed (agreement only within a factor of three of DALI)

## Inputs

- ALMA Lupus survey catalog of 12CO line fluxes
- ALMA Lupus survey catalog of 887-micron disc sizes
- Semi-analytical 12CO flux model tracks (viscous and MHD-wind scenarios, with and without CO depletion)
- Upper Sco disc sample (comparison population)

## Outputs

- Model-vs-data overlay figure of Lupus 12CO fluxes and disc sizes with viscous and MHD-wind tracks (depletion vs no-depletion)
- Sample-count table comparing detectable discs under the 12CO flux criterion vs the disc-size criterion (factor 1.3 increase for Lupus, 3.6 for Upper Sco)
- Summary table and companion figure for the Lupus condition

## How to apply

Load the Lupus survey catalog of 12CO line fluxes and 887-micron disc sizes observed with ALMA. Compute semi-analytical model 12CO fluxes for the viscous and MHD-wind evolutionary tracks, running the model twice — once with CO depletion included and once without — to bracket the initial-condition uncertainty, and overlay the tracks on the observed Lupus fluxes and sizes. Accept model–data agreement only in the CO-depletion case, since the article's proof-of-concept found models and data agree well only if CO depletion is considered. Count detectable Lupus discs twice, once with the 12CO flux criterion and once with the disc-size criterion, and verify the reported sample increase of factor 1.3 for Lupus when studying fluxes instead of sizes. Assemble the model-vs-data overlay and the sample-count comparison into a summary table and companion figure for the Lupus condition.

## Related tools

- **ALMA** (Instrument providing the observed Lupus (and Upper Sco) 12CO line fluxes and 887-micron disc sizes against which the model tracks are compared)
- **DALI** (Thermochemical model used to benchmark the semi-analytical 12CO flux computation (agreement within a factor of three))
- **COpops** (Code implementing the semi-analytical CO flux models used to synthesise disc populations for the Lupus comparison) — github.com/fzagaria/COpops

## Evaluation signals

- Model tracks and observed Lupus data agree well only in the CO-depletion case; agreement in the no-depletion run signals a misapplication
- The counted detectable Lupus sample using the 12CO flux criterion is a factor 1.3 larger than the sample using the disc-size criterion
- The semi-analytical flux model reproduces DALI thermochemical results within a factor of three
- The output includes both a model-vs-data summary table and a companion figure for the Lupus condition
- Both evolutionary scenarios (viscous and MHD-wind) are run twice each, bracketing the initial-condition uncertainty with CO depletion on/off

## Limitations

- Uncertainties on the initial conditions limited interpretation of the observations
- Available data have not given conclusive answers yet on the underlying disc evolution mechanisms: viscosity or magnetohydrodynamic (MHD) winds
- Semi-analytical fluxes agree with full physico-chemical models only within a factor of three, so precision is limited relative to DALI thermochemical modelling

## Evidence

- [abstract] Models and data agree well only if CO depletion is considered: "Models and data agree well only if CO depletion is considered"
- [abstract] Sample increase factor when using fluxes instead of sizes: "increased samples, by a factor 1.3 (Lupus) and 3.6 (Upper Sco), when studying fluxes instead of sizes"
- [readme] Semi-analytical method is fast, flexible, accurate relative to thermochemical models: "compared to more-complex physico-chemical models, we recover a good agreement within a factor of three"
- [abstract] Uncertainty in initial conditions limited the observational interpretation: "the uncertainties on the initial conditions limited our interpretation of the observations"
