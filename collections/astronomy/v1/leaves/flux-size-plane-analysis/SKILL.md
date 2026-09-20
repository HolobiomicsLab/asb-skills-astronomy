---
name: flux-size-plane-analysis
description: 'Use when reproducing or extending the Upper Sco 12CO flux-size comparison:
  you have an ALMA survey catalog of 12CO integrated fluxes and CO-disc radii and
  want to overlay semi-analytical COpops model tracks with CO depletion switched on
  and off.'
when_to_use_negative:
- When the goal is to test viscous vs MHD-wind disc evolution scenarios on currently-available
  data — the paper delegates that interpretation to Zagaria et al. (2023b) because
  uncertainties on initial conditions limited interpretation
- When only disc sizes (radii) are available without 12CO integrated fluxes, since
  the skill's core comparison is in the flux-size plane and the flux-based selection
  gain cannot be computed
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ALMA
  role: Source of the observational 12CO integrated fluxes and CO-disc radii for the
    Upper Sco and Lupus disc samples compared to the models
- name: DALI
  role: Thermochemical models used to benchmark the semi-analytical flux model, recovering
    agreement within a factor of three
- name: COpops
  role: Repository implementing the fast, flexible, semi-analytical 12CO flux model
    used to predict fluxes for each Upper Sco disc
  repo: https://github.com/fzagaria/COpops
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:6d1963f4817b7578e3011148a71c24674d1e4430c369db33506382c2972ffd1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fzagaria/COpops
  tools:
  - ALMA
  - COpops
  - DALI
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# flux-size-plane-analysis

## Summary

Reproduces the paper's comparison of semi-analytical 12CO flux models (COpops) against the Upper Sco (and Lupus) ALMA disc sample in the flux-size plane, testing that model–data agreement requires a CO-depletion prescription and quantifying the sample-gain factor of 3.6 from flux-based over size-based selection.

## When to use

Use when reproducing or extending the Upper Sco 12CO flux-size comparison: you have an ALMA survey catalog of 12CO integrated fluxes and CO-disc radii and want to overlay semi-analytical COpops model tracks with CO depletion switched on and off.

## When NOT to use

- When the goal is to test viscous vs MHD-wind disc evolution scenarios on currently-available data — the paper delegates that interpretation to Zagaria et al. (2023b) because uncertainties on initial conditions limited interpretation
- When only disc sizes (radii) are available without 12CO integrated fluxes, since the skill's core comparison is in the flux-size plane and the flux-based selection gain cannot be computed

## Inputs

- Upper Sco survey catalog of 12CO integrated fluxes and CO-disc radii (observational sample)
- Lupus survey catalog of 12CO integrated fluxes and CO-disc radii (optional secondary sample)
- Semi-analytical 12CO flux model implementation from github.com/fzagaria/COpops
- CO-depletion prescription parameters (gas-phase CO abundance reduced relative to canonical value)

## Outputs

- upper_sco_model_comparison.csv (per-disc and sample-level model–data comparison)
- upper_sco_flux_size_comparison.png (flux-size plane plot with overlaid model tracks)
- Sample-size gain factor (flux-selected vs size-selected disc counts, expected 3.6 for Upper Sco)

## How to apply

Load the Upper Sco survey catalog of 12CO integrated fluxes and CO-disc radii used by the paper as the observational sample. For each disc, compute predicted 12CO fluxes with the semi-analytical model implemented in the COpops repository, which assumes optically thick 12CO emission so that fluxes scale with disc surface area. Recompute the model with a CO-depletion prescription switched on, reducing the gas-phase CO abundance relative to the canonical value. Overlay both model tracks on the observed flux-size plane and check quantitatively — using the paper's factor-of-three agreement tolerance (the same band recovered against DALI thermochemical models) — that agreement holds only for the CO-depleted models. Finally, count discs detected/usable when selected by 12CO flux versus by disc size and verify the flux-based sample is larger by the reported factor of 3.6 for Upper Sco (1.3 for Lupus).

## Related tools

- **ALMA** (Source of the observational 12CO integrated fluxes and CO-disc radii for the Upper Sco and Lupus disc samples compared to the models)
- **DALI** (Thermochemical models used to benchmark the semi-analytical flux model, recovering agreement within a factor of three)
- **COpops** (Repository implementing the fast, flexible, semi-analytical 12CO flux model used to predict fluxes for each Upper Sco disc) — https://github.com/fzagaria/COpops

## Evaluation signals

- Model–data agreement in the flux-size plane holds only for the CO-depleted model tracks when using the paper's factor-of-three agreement tolerance as the acceptance band
- The semi-analytical model reproduces DALI thermochemical fluxes within a factor of three before comparison to data
- Count of discs usable when selected by 12CO flux divided by count selected by disc size equals ~3.6 for Upper Sco (and ~1.3 for Lupus as a cross-check)
- upper_sco_model_comparison.csv contains both per-disc and sample-level comparison records
- Flux tracks are consistent with the optically thick 12CO assumption, i.e. fluxes scale with disc surface area

## Limitations

- The available ALMA data have not given conclusive answers on the underlying disc evolution mechanisms (viscosity or MHD winds)
- Uncertainties on the initial conditions limited interpretation of the observations
- The semi-analytical model's accuracy is bounded by its agreement with more-complex physico-chemical models within a factor of three

## Evidence

- [abstract] Models and data agree only if CO depletion is considered: "Models and data agree well only if CO depletion is considered"
- [abstract] Sample gain factor 3.6 for Upper Sco: "increased samples, by a factor 1.3 (Lupus) and 3.6 (Upper Sco), when studying fluxes instead of sizes"
- [abstract] Optically thick flux scaling: "assuming that 12CO is optically thick, fluxes scale with the disc surface area"
- [abstract] Factor-of-three benchmark against DALI: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [readme] Lupus and Upper Sco comparison: "In Zagaria et al. (2023b) we discuss how these models can be used to understand better the currently-available data in Lupus and Upper Sco."
