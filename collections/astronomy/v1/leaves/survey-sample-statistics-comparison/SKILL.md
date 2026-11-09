---
name: survey-sample-statistics-comparison
description: Use when comparing semi-analytical disc-evolution model predictions against
  an ALMA disc survey catalog (flux- or size-based) to determine under which initial-condition
  assumptions (e.
when_to_use_negative:
- 'When a conclusive discrimination between viscous and MHD-wind evolution is required:
  the available ALMA data have not given conclusive answers yet on the underlying
  disc evolution mechanisms.'
- 'When high-precision per-disc physico-chemical modelling is needed rather than population-level
  comparison: the semi-analytical method recovers agreement with DALI only within
  a factor of three.'
- When initial conditions are tightly constrained; the paper notes uncertainties on
  the initial conditions limited the interpretation of the observations.
tools:
- name: ALMA
  role: Source of the survey data (12CO line fluxes and 887-micron disc sizes for
    Lupus and Upper Sco) compared against the semi-analytical model tracks.
- name: DALI
  role: Thermochemical reference models used to benchmark the semi-analytical 12CO
    flux method (agreement within a factor of three) before the population comparison.
- name: COpops
  role: Code implementing the semi-analytical disc CO flux models used to synthesise
    populations and compare with survey data.
  repo: github.com/fzagaria/COpops
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:3ef2a12f39143f1b84b6a4f8bb1af74af59d74e4acef3ea31b4a828a7aa735a2
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# survey-sample-statistics-comparison

## Summary

Overlay semi-analytical 12CO flux model tracks (viscous and MHD-wind evolutionary scenarios, run with and without CO depletion) on observed ALMA survey samples (e.g., Lupus 12CO line fluxes and 887-micron disc sizes) to test model–data agreement and quantify how usable sample sizes change between observables.

## When to use

Use when comparing semi-analytical disc-evolution model predictions against an ALMA disc survey catalog (flux- or size-based) to determine under which initial-condition assumptions (e.g., CO depletion) models and data agree, and to count how many discs are detectable under different observable criteria.

## When NOT to use

- When a conclusive discrimination between viscous and MHD-wind evolution is required: the available ALMA data have not given conclusive answers yet on the underlying disc evolution mechanisms.
- When high-precision per-disc physico-chemical modelling is needed rather than population-level comparison: the semi-analytical method recovers agreement with DALI only within a factor of three.
- When initial conditions are tightly constrained; the paper notes uncertainties on the initial conditions limited the interpretation of the observations.

## Inputs

- Survey catalog of ALMA 12CO line fluxes (e.g., Lupus, Upper Sco)
- Survey catalog of ALMA 887-micron disc sizes
- Semi-analytical 12CO flux model outputs for viscous and MHD-wind evolutionary tracks, run with and without CO depletion
- Detectability thresholds defining the usable sample under the flux criterion and the size criterion

## Outputs

- Overlay figure of semi-analytical 12CO flux model tracks on observed survey fluxes and sizes
- Count of detectable discs under the 12CO flux criterion vs the disc-size criterion (e.g., factor 1.3 sample increase for Lupus)
- Summary table of model-vs-data agreement under each initial-condition assumption (CO depletion vs no depletion)
- Statement of whether models and data agree only under CO depletion

## How to apply

Load the survey catalog of ALMA observables (for Lupus: 12CO line fluxes and 887-micron disc sizes). Compute semi-analytical 12CO fluxes for both the viscous and MHD-wind evolutionary tracks, running the model twice — once with CO depletion included and once without — to bracket the initial-condition uncertainty. Overlay the model tracks on the observed fluxes and sizes and check agreement: in the Zagaria et al. benchmark, models and data agree well only in the CO-depletion case. Count detectable discs twice, once using the 12CO flux criterion and once using the disc-size criterion, to quantify sample-size differences (reported as a factor 1.3 for Lupus and 3.6 for Upper Sco when using fluxes instead of sizes). Assemble the model-vs-data comparison and the sample-count comparison into a summary table and companion figure. Sanity-check the semi-analytical model against DALI thermochemical models beforehand, expecting agreement within a factor of three.

## Related tools

- **ALMA** (Source of the survey data (12CO line fluxes and 887-micron disc sizes for Lupus and Upper Sco) compared against the semi-analytical model tracks.)
- **DALI** (Thermochemical reference models used to benchmark the semi-analytical 12CO flux method (agreement within a factor of three) before the population comparison.)
- **COpops** (Code implementing the semi-analytical disc CO flux models used to synthesise populations and compare with survey data.) — github.com/fzagaria/COpops

## Examples

```
python -m copops.run --tracks viscous,mhd-wind --co-depletion true,false --survey lupus --out model_vs_lupus
```

## Evaluation signals

- Model tracks overlay the observed Lupus fluxes and sizes well only in the CO-depletion run; the no-depletion run should show disagreement.
- The semi-analytical 12CO fluxes agree with DALI thermochemical models within a factor of three.
- Detectable-sample counts computed twice (12CO flux criterion vs disc-size criterion) reproduce the reported sample increase factors: 1.3 for Lupus and 3.6 for Upper Sco.
- Both viscous and MHD-wind tracks are present in the comparison, with CO-depletion vs no-depletion variants bracketing the initial-condition uncertainty.
- The comparison is delivered as a summary table plus a companion figure per region/condition.

## Limitations

- Models and data agree well only if CO depletion is considered; interpreting data without this initial-condition treatment is unreliable.
- Uncertainties on the initial conditions limited the interpretation of the observations.
- The available ALMA data have not given conclusive answers yet on the underlying disc evolution mechanisms: viscosity or magnetohydrodynamic (MHD) winds.
- The semi-analytical method is accurate only within a factor of three relative to full physico-chemical (DALI) models.

## Evidence

- [abstract] Models and data agree only if CO depletion is considered: "Models and data agree well only if CO depletion is considered"
- [abstract] Sample increase factors: "increased samples, by a factor 1.3 (Lupus) and 3.6 (Upper Sco), when studying fluxes instead of sizes"
- [abstract] DALI benchmark threshold: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [intro] Model comparison purpose: "In Zagaria et al. (2023b) we discuss how these models can be used to understand better the currently-available data in Lupus and Upper Sco."
- [abstract] Missing conclusive answer: "the available data have not given conclusive answers yet on the underlying disc evolution mechanisms: viscosity or magnetohydrodynamic (MHD) winds"
- [readme] Repository purpose: "Our method shows how powerful CO fluxes can be when studying disc evolution!"
