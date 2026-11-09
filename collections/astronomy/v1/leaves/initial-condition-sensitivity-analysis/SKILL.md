---
name: initial-condition-sensitivity-analysis
description: Use when model–data agreement for CO fluxes of a disc sample is inconclusive
  and you need to test whether uncertainties in the model's initial conditions (disc
  initial mass and radius) limit the interpretation of the observations.
when_to_use_negative:
- When a detailed physico-chemical treatment of a single disc is needed — the semi-analytical
  model is a fast, flexible alternative but DALI thermochemical models are the benchmark
  of reference
- When the model–data comparison is not limited by initial-condition uncertainties
  (e.g. when CO depletion or observational noise dominates the disagreement)
- When the target flux is not accessible in the semi-analytical 12CO framework (e.g.
  optically thin tracers rather than optically thick 12CO)
tools:
- name: COpops
  role: Provides the semi-analytical 12CO flux model and the parameterized initial
    disc mass and radius ranges used to build the sensitivity grid
  repo: github.com/fzagaria/COpops
- name: ALMA
  role: Source of the observed 12CO fluxes of the Lupus and Upper Sco disc samples
    against which grid-point predictions are compared
- name: DALI
  role: Thermochemical models providing the benchmark (agreement within a factor of
    three) for validating the semi-analytical flux predictions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:fde414b227d3217d530bda5573d32d89a1f2fa8b4c57bef88afa48ef655f8f20
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
yes_rate: 0.6666666666666666
n_claims: 6
---

# initial-condition-sensitivity-analysis

## Summary

Quantifies how uncertainties in protoplanetary-disc initial conditions (initial disc mass and radius) propagate into semi-analytical 12CO flux predictions, by running a grid of COpops initial-condition combinations and comparing predicted fluxes against observed ALMA fluxes of the Lupus and Upper Sco disc samples.

## When to use

Use when model–data agreement for CO fluxes of a disc sample is inconclusive and you need to test whether uncertainties in the model's initial conditions (disc initial mass and radius) limit the interpretation of the observations.

## When NOT to use

- When a detailed physico-chemical treatment of a single disc is needed — the semi-analytical model is a fast, flexible alternative but DALI thermochemical models are the benchmark of reference
- When the model–data comparison is not limited by initial-condition uncertainties (e.g. when CO depletion or observational noise dominates the disagreement)
- When the target flux is not accessible in the semi-analytical 12CO framework (e.g. optically thin tracers rather than optically thick 12CO)

## Inputs

- Initial disc mass and radius ranges parameterized in the COpops package
- Disc ages of the target samples (Lupus and Upper Sco)
- Observed ALMA 12CO fluxes of the Lupus and Upper Sco disc samples

## Outputs

- Per-grid-point predicted 12CO fluxes across the initial-condition grid
- Spread summary statistics (factor-of range and percentile spread of predicted fluxes and of model–data agreement)
- Sensitivity table written to initial_condition_sensitivity.csv

## How to apply

Load the initial disc mass and radius ranges parameterized in the COpops package — these are the initial conditions the paper flagged as limiting interpretation. Construct a grid of initial-condition combinations spanning those ranges at the disc ages of the samples being compared (here Lupus and Upper Sco). For each grid point, run the semi-analytical 12CO flux model (fast, flexible alternative to computationally-expensive thermochemical models like DALI) to obtain predicted fluxes, and compare each prediction against the observed ALMA 12CO fluxes. Compute spread statistics across the grid — the factor-of range and percentile spread of predicted fluxes and of the model–data agreement. If the recovered agreement stays within the benchmark that the semi-analytical model matches DALI thermochemical models within a factor of three, the interpretation is constrained; wide spreads indicate initial conditions dominate the uncertainty. Write per-grid-point fluxes and the spread summary to a sensitivity table.

## Related tools

- **COpops** (Provides the semi-analytical 12CO flux model and the parameterized initial disc mass and radius ranges used to build the sensitivity grid) — github.com/fzagaria/COpops
- **ALMA** (Source of the observed 12CO fluxes of the Lupus and Upper Sco disc samples against which grid-point predictions are compared)
- **DALI** (Thermochemical models providing the benchmark (agreement within a factor of three) for validating the semi-analytical flux predictions)

## Evaluation signals

- The sensitivity table covers the full parameterized initial-condition grid (all combinations of initial disc mass and radius ranges at the Lupus and Upper Sco ages)
- Recovered model–data agreement is reported against the benchmark that the semi-analytical model agrees with DALI thermochemical models within a factor of three
- Spread statistics include both a factor-of range and percentile spread of predicted fluxes and of model–data agreement
- Per-grid-point fluxes plus the spread summary are written to initial_condition_sensitivity.csv
- Every grid-point prediction is individually compared against the observed ALMA 12CO fluxes, not just a fiducial model

## Limitations

- In the paper's proof of concept, uncertainties on the initial conditions limited the interpretation of the Lupus and Upper Sco observations — the comparison could not be conclusively interpreted
- The semi-analytical model is only accurate to within a factor of three relative to DALI thermochemical models, so sensitivity spreads smaller than this benchmark are not meaningful
- The method assumes 12CO is optically thick, with fluxes scaling with the disc surface area
- Models and data agree only if CO depletion is considered, so sensitivity conclusions depend on the CO-depletion assumption

## Evidence

- [abstract] initial-condition uncertainties limit interpretation: "the uncertainties on the initial conditions limited our interpretation of the observations"
- [abstract] factor-of-three benchmark vs DALI: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [abstract] CO depletion required for agreement: "Models and data agree well only if CO depletion is considered"
- [readme] COpops as fast, flexible semi-analytical alternative: "an alternative method based on modelling semi-analytically the disc CO fluxes. This method is fast, flexible and accurate"
- [abstract] 12CO fluxes as disc-evolution observables: "assuming that 12CO is optically thick, fluxes scale with the disc surface area"
