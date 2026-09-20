---
name: 12co-line-flux-modelling
description: Use when synthesising 12CO fluxes for populations of protoplanetary discs
  (e.g. Lupus, Upper Sco) where running thermochemical codes per disc is prohibitively
  slow, or when tracing disc surface-area evolution from optically thick CO fluxes.
when_to_use_negative:
- When a full physico-chemical treatment of the disc (thermochemical structure, detailed
  CO chemistry) is required — DALI-class models, not the semi-analytical flux model,
  are the right tool
- When CO is expected to be optically thin or severely depleted such that the optically-thick
  flux–surface-area scaling no longer holds
- When interpreting flux ratios for conclusions about viscosity vs MHD-wind disc evolution
  without acknowledging that initial-condition uncertainties limited interpretation
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3934
tools:
- name: COpops
  role: Python package implementing the semi-analytical 12CO flux model, disc parameter
    grids, and initial-condition parameterisation
  repo: github.com/fzagaria/COpops
- name: DALI
  role: Thermochemical reference model used to benchmark the semi-analytical 12CO
    fluxes (agreement within a factor of three)
- name: ALMA
  role: Source of observed 12CO fluxes for Lupus and Upper Sco disc samples used in
    the proof-of-concept comparison
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:d759a95c01e49b1b80cac34cf5f1813e868f796efb648884c30f5c70e4ac16e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA
  - COpops
  - DALI
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# 12co-line-flux-modelling

## Summary

A semi-analytical method to compute 12CO line fluxes of protoplanetary discs using power-law surface-density and temperature profiles, avoiding computationally-expensive thermochemical modelling (DALI) for hundreds of discs. It is the core capability of the COpops package for testing disc evolution scenarios with ALMA CO fluxes.

## When to use

Use when synthesising 12CO fluxes for populations of protoplanetary discs (e.g. Lupus, Upper Sco) where running thermochemical codes per disc is prohibitively slow, or when tracing disc surface-area evolution from optically thick CO fluxes.

## When NOT to use

- When a full physico-chemical treatment of the disc (thermochemical structure, detailed CO chemistry) is required — DALI-class models, not the semi-analytical flux model, are the right tool
- When CO is expected to be optically thin or severely depleted such that the optically-thick flux–surface-area scaling no longer holds
- When interpreting flux ratios for conclusions about viscosity vs MHD-wind disc evolution without acknowledging that initial-condition uncertainties limited interpretation

## Inputs

- Disc parameter grids (surface-density and characteristic-radius combinations) shipped with the COpops package
- Initial disc mass and radius ranges for the sample population (Lupus, Upper Sco) at their respective ages
- Observed ALMA 12CO line fluxes of the Lupus and Upper Sco disc samples

## Outputs

- Per-grid-point 12CO line fluxes keyed by disc parameters (e.g. co_flux_table.csv)
- Initial-condition sensitivity table with per-grid-point fluxes and spread statistics (factor-of range, percentile spread) of model–data agreement (e.g. initial_condition_sensitivity.csv)

## How to apply

Represent each disc with power-law surface-density and characteristic-radius parameters from the COpops grids, evaluate the semi-analytical disc structure, and integrate the emergent 12CO emission over the disc surface area under the optically thick assumption — since fluxes scale with the disc surface area. Run this per grid point instead of invoking DALI, then assemble per-disc fluxes into a table keyed by disc parameters (e.g. co_flux_table.csv). Validate by comparing against DALI benchmarks: accept results when agreement falls within a factor of three. For population studies, grid over initial-condition combinations (initial disc mass and radius ranges) at the Lupus/Upper Sco disc ages and compute spread statistics of predicted fluxes against ALMA observations.

## Related tools

- **COpops** (Python package implementing the semi-analytical 12CO flux model, disc parameter grids, and initial-condition parameterisation) — github.com/fzagaria/COpops
- **DALI** (Thermochemical reference model used to benchmark the semi-analytical 12CO fluxes (agreement within a factor of three))
- **ALMA** (Source of observed 12CO fluxes for Lupus and Upper Sco disc samples used in the proof-of-concept comparison)

## Examples

```
pip install COpops; then load disc parameter grids from github.com/fzagaria/COpops and evaluate the semi-analytical 12CO flux per grid point into co_flux_table.csv
```

## Evaluation signals

- Semi-analytical 12CO fluxes agree with DALI thermochemical benchmarks within a factor of three
- The flux table covers the full parameterised grid of disc surface-density and characteristic-radius combinations
- Sensitivity analysis spans the full initial disc mass and radius ranges at the Lupus and Upper Sco ages
- Under the optically thick assumption, fluxes scale with the disc surface area — check flux ratios against surface-area ratios of the models
- Reproduce the reference flux values reported by the COpops package/article for representative disc models

## Limitations

- Agreement with physico-chemical models is only within a factor of three, not exact
- Uncertainties on initial conditions limited the interpretation of the Lupus/Upper Sco observations — no conclusive answers on viscosity vs MHD winds
- Models and data agree well only if CO depletion is considered
- Optically thick 12CO assumption is required for the flux–surface-area scaling

## Evidence

- [abstract] benchmark accuracy: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [abstract] flux-area scaling: "assuming that 12CO is optically thick, fluxes scale with the disc surface area"
- [intro] motivation: "Here we introduce an alternative method based on modelling semi-analytically the disc CO fluxes. This method is fast, flexible and accurate"
- [abstract] initial-condition limits: "the uncertainties on the initial conditions limited our interpretation of the observations"
- [abstract] CO depletion requirement: "Models and data agree well only if CO depletion is considered"
- [readme] package scope: "running computationally-expensive thermochemical models for hundreds of discs can be prohibitively time-consuming"
