---
name: disc-evolution-model-data-comparison
description: Use when ALMA survey catalogs of 12CO line fluxes and 887-micron disc
  sizes need to be overlaid with disc-evolution model tracks to test viscous vs MHD-wind
  scenarios, or when deciding whether to select samples by flux or size criteria.
when_to_use_negative:
- The goal is conclusive determination of the underlying disc evolution mechanism
  (viscosity or MHD winds) — the article states available ALMA data have not given
  conclusive answers yet
- Computationally-expensive thermochemical modelling is feasible for the full disc
  population — the semi-analytical method exists precisely because running DALI-like
  models for hundreds of discs is prohibitively time-consuming
tools:
- name: ALMA
  role: Source of the 12CO line flux and 887-micron disc-size observations of Lupus
    and Upper Sco used as the data side of the comparison
- name: DALI
  role: Thermochemical models used to benchmark the semi-analytical 12CO flux model,
    recovering agreement within a factor of three
- name: COpops
  role: Repository providing the code to synthesise disc populations and compute semi-analytical
    12CO fluxes for the comparison
  repo: github.com/fzagaria/COpops
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:2cbe4a3cbc119c53d36cf8ced733ac041482f5c1bc171af05467723fc429944a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA
  - COpops
  - DALI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# disc-evolution-model-data-comparison

## Summary

Compares semi-analytical 12CO flux models (viscous and MHD-wind evolutionary tracks) against ALMA-observed disc samples such as Lupus and Upper Sco, testing whether models and data agree only when CO depletion is included, and quantifying how sample sizes change when fluxes are used instead of disc sizes.

## When to use

Use when ALMA survey catalogs of 12CO line fluxes and 887-micron disc sizes need to be overlaid with disc-evolution model tracks to test viscous vs MHD-wind scenarios, or when deciding whether to select samples by flux or size criteria.

## When NOT to use

- The goal is conclusive determination of the underlying disc evolution mechanism (viscosity or MHD winds) — the article states available ALMA data have not given conclusive answers yet
- Computationally-expensive thermochemical modelling is feasible for the full disc population — the semi-analytical method exists precisely because running DALI-like models for hundreds of discs is prohibitively time-consuming

## Inputs

- Survey catalog of ALMA 12CO line fluxes for discs in a star-forming region (e.g. Lupus)
- ALMA-observed 887-micron disc sizes for the same sample
- Semi-analytical 12CO flux model tracks for viscous and MHD-wind evolutionary scenarios
- DALI thermochemical model results for benchmarking

## Outputs

- Overlay figure of model tracks on observed fluxes and disc sizes
- Summary table of model-vs-data agreement per evolutionary scenario and CO-depletion condition
- Sample-count comparison: number of detectable discs by flux criterion vs size criterion (e.g. factor 1.3 for Lupus)

## How to apply

Load the survey catalog (e.g. Lupus 12CO line fluxes and 887-micron disc sizes from ALMA) and compute semi-analytical model 12CO fluxes for both viscous and MHD-wind evolutionary tracks, running the model twice — once with CO depletion included and once without — to bracket the initial-condition uncertainty. Overlay the model tracks on the observed flux-vs-size distribution and check that models and data agree well only in the CO-depletion case. Then count detectable discs twice, once with a 12CO flux criterion and once with a disc-size criterion, to verify the reported sample increase (factor 1.3 for Lupus, 3.6 for Upper Sco) when studying fluxes instead of sizes. Validate the semi-analytical fluxes against DALI thermochemical models, expecting agreement within a factor of three. Assemble the model-vs-data comparison and sample-count comparison into a summary table and companion figure.

## Related tools

- **ALMA** (Source of the 12CO line flux and 887-micron disc-size observations of Lupus and Upper Sco used as the data side of the comparison)
- **DALI** (Thermochemical models used to benchmark the semi-analytical 12CO flux model, recovering agreement within a factor of three)
- **COpops** (Repository providing the code to synthesise disc populations and compute semi-analytical 12CO fluxes for the comparison) — github.com/fzagaria/COpops

## Evaluation signals

- Model tracks overlay onto observed Lupus fluxes and sizes only in the CO-depletion case; agreement should fail when CO depletion is omitted
- Semi-analytical 12CO fluxes reproduce DALI thermochemical model fluxes within a factor of three
- Counting detectable Lupus discs with a 12CO flux criterion vs a disc-size criterion yields a sample increase of factor 1.3 (3.6 for Upper Sco)
- Both viscous and MHD-wind tracks are run under both initial-condition brackets (with and without CO depletion), giving a four-track comparison grid

## Limitations

- Uncertainties on the initial conditions limited the interpretation of the observations
- Available ALMA data have not given conclusive answers on the underlying disc evolution mechanisms (viscosity vs MHD winds)
- The semi-analytical model agreement is only within a factor of three, not exact, relative to thermochemical models

## Evidence

- [abstract] Models and data agree only if CO depletion is considered: "Models and data agree well only if CO depletion is considered"
- [abstract] Sample size increase factors: "increased samples, by a factor 1.3 (Lupus) and 3.6 (Upper Sco), when studying fluxes instead of sizes"
- [abstract] Semi-analytical vs DALI benchmark tolerance: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [readme] Motivation for semi-analytical method: "running computationally-expensive thermochemical models for hundreds of discs can be prohibitively time-consuming"
- [abstract] Initial-condition uncertainty limitation: "the uncertainties on the initial conditions limited our interpretation of the observations"
