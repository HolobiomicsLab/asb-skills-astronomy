---
name: co-depletion-modelling
description: Use when comparing semi-analytical 12CO flux model predictions to ALMA
  survey observations (e.g. Upper Sco or Lupus fluxes and CO-disc radii) and model–data
  mismatch suggests volatiles are depleted; also when quantifying how CO depletion
  shifts model tracks in the flux–size plane.
when_to_use_negative:
- When a fully self-consistent physico-chemical treatment of CO chemistry and thermal
  balance is required — use thermochemical codes like DALI instead; the semi-analytical
  model is only benchmarked against DALI to within a factor of three
- When interpreting isotopologue emission for which the optically thick 12CO assumption
  (flux scaling with disc surface area) does not hold
edam_operation: http://edamontology.org/operation_2426
tools:
- name: ALMA
  role: Source of the observed 12CO integrated fluxes and CO-disc radii for the Upper
    Sco (and Lupus) disc samples against which the CO-depleted model tracks are compared
- name: COpops
  role: Implements the semi-analytical 12CO flux model in which the CO-depletion prescription
    is switched on/off; used to generate model tracks in the flux–size plane
  repo: github.com/fzagaria/COpops
- name: DALI
  role: Thermochemical reference model used to benchmark the semi-analytical fluxes,
    recovering agreement within a factor of three
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:b3327f0c35cbabb5667fc75e593afda1a5a25b6aa24a58e603438a4cb4f140fe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA
  - COpops
  - DALI
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# co-depletion-modelling

## Summary

A modelling skill for computing 12CO fluxes from protoplanetary disc populations with a CO-depletion prescription switched on, so that gas-phase CO abundance is reduced relative to the canonical value. Models and Upper Sco/Lupus data agree well only if CO depletion is considered, making this the key model configuration for data comparison.

## When to use

Use when comparing semi-analytical 12CO flux model predictions to ALMA survey observations (e.g. Upper Sco or Lupus fluxes and CO-disc radii) and model–data mismatch suggests volatiles are depleted; also when quantifying how CO depletion shifts model tracks in the flux–size plane.

## When NOT to use

- When a fully self-consistent physico-chemical treatment of CO chemistry and thermal balance is required — use thermochemical codes like DALI instead; the semi-analytical model is only benchmarked against DALI to within a factor of three
- When interpreting isotopologue emission for which the optically thick 12CO assumption (flux scaling with disc surface area) does not hold

## Inputs

- Survey catalog of 12CO integrated fluxes for a disc sample (e.g. Upper Sco ALMA observations)
- Observed CO-disc radii (sizes) for the same sample
- Semi-analytical 12CO flux model from the COpops repository, with CO-depletion prescription switch

## Outputs

- Per-disc and sample-level model–data comparison table (e.g. upper_sco_model_comparison.csv)
- Flux–size comparison plot with overlaid canonical and CO-depleted model tracks (e.g. upper_sco_flux_size_comparison.png)
- Agreement verdict: whether data agree with models within the factor-of-three tolerance only under CO depletion

## How to apply

Load the survey catalog of 12CO integrated fluxes and CO-disc radii (e.g. the Upper Sco sample observed with ALMA), then compute predicted 12CO fluxes for each disc with the semi-analytical model in the COpops repository, which assumes optically thick 12CO emission so fluxes scale with the disc surface area. Repeat the computation with the CO-depletion prescription enabled, reducing the gas-phase CO abundance relative to the canonical value. Overlay both model tracks (canonical and CO-depleted) on the observed flux–size plane and accept agreement only when model and data agree within the paper's factor-of-three tolerance, which is the same accuracy the semi-analytical model recovers against DALI thermochemical benchmarks. Record per-disc and sample-level comparisons (e.g. upper_sco_model_comparison.csv) and the flux–size plot. Rationale: the article finds models and data agree well only when CO depletion is included, and computationally cheap semi-analytical fluxes avoid expensive thermochemical runs for hundreds of discs.

## Related tools

- **ALMA** (Source of the observed 12CO integrated fluxes and CO-disc radii for the Upper Sco (and Lupus) disc samples against which the CO-depleted model tracks are compared)
- **COpops** (Implements the semi-analytical 12CO flux model in which the CO-depletion prescription is switched on/off; used to generate model tracks in the flux–size plane) — github.com/fzagaria/COpops
- **DALI** (Thermochemical reference model used to benchmark the semi-analytical fluxes, recovering agreement within a factor of three)

## Examples

```
git clone https://github.com/fzagaria/COpops
```

## Evaluation signals

- Model–data agreement in the flux–size plane is achieved within the paper's factor-of-three tolerance only for CO-depleted model tracks; canonical-abundance tracks should show disagreement
- The semi-analytical fluxes reproduce DALI thermochemical predictions within a factor of three (the stated benchmark accuracy)
- Flux-selected sample of Upper Sco discs is larger than the size-selected sample by the reported factor of 3.6
- Per-disc outputs (upper_sco_model_comparison.csv) contain one row per Upper Sco disc with observed flux/radius and predicted fluxes for both model configurations
- Plot (upper_sco_flux_size_comparison.png) overlays observed points and both model tracks on the flux–size plane

## Limitations

- Uncertainties on the initial conditions limited interpretation of the observations
- Agreement is only accurate to within a factor of three relative to full thermochemical models
- The method assumes optically thick 12CO emission, so fluxes simply scale with the disc surface area

## Evidence

- [abstract] Models and data agree only if CO depletion is considered: "Models and data agree well only if CO depletion is considered"
- [abstract] Semi-analytical model benchmark accuracy: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [abstract] Optically thick flux scaling: "assuming that 12CO is optically thick, fluxes scale with the disc surface area"
- [abstract] Sample size increase factor for Upper Sco: "increased samples, by a factor 1.3 (Lupus) and 3.6 (Upper Sco), when studying fluxes instead of sizes"
- [readme] Method motivation and speed: "running computationally-expensive thermochemical models for hundreds of discs can be prohibitively time-consuming"
