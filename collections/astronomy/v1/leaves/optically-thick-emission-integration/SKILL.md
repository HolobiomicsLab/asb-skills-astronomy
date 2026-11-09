---
name: optically-thick-emission-integration
description: Use when you need fast 12CO flux predictions for hundreds of protoplanetary
  discs (parameter grids of surface density and characteristic radius) and can accept
  optically thick, factor-of-three accuracy instead of a thermochemical model.
when_to_use_negative:
- When sub-factor-of-three accuracy is required for individual disc structures — the
  semi-analytical model is only benchmarked to agree with DALI within a factor of
  three.
- When the line of interest is optically thin, since the method's flux–surface-area
  scaling rests on the 12CO optically thick assumption.
- When detailed thermochemical/physical structure (e.g. CO abundance vertical distribution
  without CO depletion) must be resolved rather than population-level fluxes.
tools:
- name: COpops
  role: Python package providing the semi-analytical disc structure and representative
    parameter grids used to compute optically thick 12CO fluxes
  repo: github.com/fzagaria/COpops
- name: DALI
  role: Thermochemical reference model against which the semi-analytical optically
    thick fluxes are benchmarked
- name: ALMA
  role: Source of the observed disc samples (Lupus, Upper Sco) that the computed CO
    fluxes are compared against
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:353fe90814cdf6f1d77c259dce3e39a2ef4efae3897cd2d04b768664e6ce4017
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

# optically-thick-emission-integration

## Summary

Computes 12CO line fluxes from a semi-analytical disc model (power-law surface-density and temperature profiles) by integrating emergent optically thick emission over the disc surface area, as implemented in the COpops package, avoiding full DALI thermochemical runs.

## When to use

Use when you need fast 12CO flux predictions for hundreds of protoplanetary discs (parameter grids of surface density and characteristic radius) and can accept optically thick, factor-of-three accuracy instead of a thermochemical model.

## When NOT to use

- When sub-factor-of-three accuracy is required for individual disc structures — the semi-analytical model is only benchmarked to agree with DALI within a factor of three.
- When the line of interest is optically thin, since the method's flux–surface-area scaling rests on the 12CO optically thick assumption.
- When detailed thermochemical/physical structure (e.g. CO abundance vertical distribution without CO depletion) must be resolved rather than population-level fluxes.

## Inputs

- Disc parameter grid (surface-density profile parameters and characteristic radii) as shipped with COpops
- Power-law surface-density and temperature profile parameters per disc

## Outputs

- Per-grid-point 12CO line fluxes assembled into a single table keyed by disc parameters (e.g. co_flux_table.csv)

## How to apply

For each disc in the parameter grid, build the semi-analytical disc structure in COpops using power-law surface-density and temperature profiles rather than running DALI. Assume 12CO is optically thick, so the emergent line flux scales with the disc surface area; integrate the emergent emission over that area to obtain the flux per grid point. Assemble per-grid-point fluxes into a single table keyed by the disc parameters (e.g. co_flux_table.csv). Validate the computed fluxes against DALI thermochemical benchmarks, where the semi-analytical model recovers agreement within a factor of three. When comparing to Lupus or Upper Sco observations, remember that models and data agree only if CO depletion is considered.

## Related tools

- **COpops** (Python package providing the semi-analytical disc structure and representative parameter grids used to compute optically thick 12CO fluxes) — github.com/fzagaria/COpops
- **DALI** (Thermochemical reference model against which the semi-analytical optically thick fluxes are benchmarked)
- **ALMA** (Source of the observed disc samples (Lupus, Upper Sco) that the computed CO fluxes are compared against)

## Evaluation signals

- Computed 12CO fluxes fall within a factor of three of reference DALI thermochemical benchmarks.
- Fluxes scale monotonically with the disc surface area under the optically thick assumption.
- Output table has one row per grid point, keyed by the disc surface-density and characteristic-radius parameters.
- Flux comparison with Lupus/Upper Sco data is only credible when CO depletion is included; absence of depletion should flag the comparison.

## Limitations

- Accuracy is limited to agreement within a factor of three with full thermochemical models.
- Uncertainties on initial conditions limit interpretation of the observations.
- Available ALMA data have not given conclusive answers on disc evolution mechanisms (viscosity vs MHD winds).

## Evidence

- [abstract] Flux–surface-area scaling: "assuming that 12CO is optically thick, fluxes scale with the disc surface area"
- [abstract] Benchmark accuracy: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [intro] Model purpose: "Here we introduce an alternative method based on modelling semi-analytically the disc CO fluxes. This method is fast, flexible and accurate"
- [abstract] CO depletion requirement: "Models and data agree well only if CO depletion is considered"
- [readme] README method statement: "This method is fast, flexible and accurate: compared to more-complex physico-chemical models, we recover a good agreement within a factor of three."
