---
name: model-data-agreement-quantification
description: Use when comparing population-level model predictions of 12CO fluxes
  against observed ALMA fluxes (e.g. Lupus or Upper Sco samples) and you need to quantify
  model-data agreement and its sensitivity to initial-condition uncertainties.
when_to_use_negative:
- When computationally-expensive thermochemical models (e.g. DALI) are needed for
  detailed physico-chemical modelling of individual discs — the semi-analytical approach
  trades chemical detail for speed (agreement only within a factor of three).
- When the sample has disc sizes but no measured 12CO fluxes and flux-based sample
  growth (1.3x Lupus, 3.6x Upper Sco) is not achievable from the available data.
edam_operation: http://edamontology.org/operation_0306
tools:
- name: COpops
  role: Semi-analytical 12CO flux model and initial-condition parameterization used
    to generate per-grid-point flux predictions
  repo: github.com/fzagaria/COpops
- name: DALI
  role: Thermochemical reference models against which the semi-analytical fluxes are
    benchmarked (agreement within a factor of three)
- name: ALMA
  role: Source of the observed 12CO fluxes of the Lupus and Upper Sco disc samples
    used in the model-data comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:f7a2ef4de8ddf2991bf31181b19697d9562f098465d20c7352a8d4e65c854939
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA
  - COpops
  - DALI
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# model-data-agreement-quantification

## Summary

Quantifies how well semi-analytical 12CO flux predictions agree with observed ALMA fluxes of disc populations (Lupus, Upper Sco), including how the agreement spreads when model initial conditions (disc mass, radius) are varied. It provides the benchmarked factor-of-three accuracy framing used to judge whether a fast CO-flux model is trustworthy against both DALI thermochemical models and real data.

## When to use

Use when comparing population-level model predictions of 12CO fluxes against observed ALMA fluxes (e.g. Lupus or Upper Sco samples) and you need to quantify model-data agreement and its sensitivity to initial-condition uncertainties.

## When NOT to use

- When computationally-expensive thermochemical models (e.g. DALI) are needed for detailed physico-chemical modelling of individual discs — the semi-analytical approach trades chemical detail for speed (agreement only within a factor of three).
- When the sample has disc sizes but no measured 12CO fluxes and flux-based sample growth (1.3x Lupus, 3.6x Upper Sco) is not achievable from the available data.

## Inputs

- Initial disc mass and radius ranges parameterized in the COpops package
- Disc region ages (Lupus, Upper Sco)
- Observed ALMA 12CO fluxes for the disc sample

## Outputs

- Per-grid-point predicted 12CO fluxes
- Spread summary of predicted fluxes and model-data agreement (factor-of range, percentile spread)
- initial_condition_sensitivity.csv

## How to apply

Run the COpops semi-analytical 12CO flux model over a grid of initial disc masses and radii parameterized in the package, for the ages of the target region (Lupus, Upper Sco). Compare each grid-point predicted flux against the observed ALMA 12CO fluxes of the sample, and compute spread statistics (factor-of ranges and percentile spreads) of predicted fluxes and of model-data agreement across the grid. Judge correctness against the paper's benchmark that the semi-analytical model recovers agreement with DALI thermochemical models within a factor of three. Interpret the observed fluxes only within the spread imposed by initial-condition uncertainties, since the paper found these uncertainties limited the interpretation of the observations.

## Related tools

- **COpops** (Semi-analytical 12CO flux model and initial-condition parameterization used to generate per-grid-point flux predictions) — github.com/fzagaria/COpops
- **DALI** (Thermochemical reference models against which the semi-analytical fluxes are benchmarked (agreement within a factor of three))
- **ALMA** (Source of the observed 12CO fluxes of the Lupus and Upper Sco disc samples used in the model-data comparison)

## Evaluation signals

- The sensitivity table covers the full parameterized initial-condition grid (mass x radius combinations) for the Lupus and Upper Sco ages.
- Recovered model-data agreement is reported against the factor-of-three benchmark from the DALI comparison.
- Spread statistics (factor-of range and percentile spread) of predicted fluxes and of model-data agreement are computed across the grid, not just at a single fiducial point.
- Output CSV contains per-grid-point fluxes plus the spread summary (initial_condition_sensitivity.csv).
- Interpretation of Lupus/Upper Sco data explicitly accounts for initial-condition uncertainty rather than claiming a constrained disc-evolution verdict.

## Limitations

- Uncertainties on the model initial conditions limited the interpretation of the observations, so the Lupus/Upper Sco comparison could not be conclusively interpreted.
- The semi-analytical model agrees with DALI only within a factor of three, so sub-factor-of-three differences are not meaningful.
- Models and data agree well only if CO depletion is considered — agreement without CO depletion should not be over-interpreted.
- Available data have not given conclusive answers on the underlying disc evolution mechanisms (viscosity vs MHD winds).

## Evidence

- [abstract] DALI benchmark factor-of-three: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [abstract] initial conditions limit interpretation: "the uncertainties on the initial conditions limited our interpretation of the observations"
- [abstract] proof-of-concept comparison samples: "As a proof of concept we compared our models with Lupus and Upper Sco data"
- [abstract] CO depletion required for agreement: "Models and data agree well only if CO depletion is considered"
- [readme] README accuracy claim: "This method is fast, flexible and accurate: compared to more-complex physico-chemical models, we recover a good agreement within a factor of three."
