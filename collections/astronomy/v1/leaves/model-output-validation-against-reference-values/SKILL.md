---
name: model-output-validation-against-reference-values
description: Use when a fast surrogate or semi-analytical model (e.g. COpops 12CO
  fluxes) must be checked against a reference set of expensive physical model outputs
  (e.g. DALI thermochemical fluxes) before population-scale use.
when_to_use_negative:
- When sub-factor-of-three precision is required, e.g. individual-disc physico-chemical
  characterisation rather than population-level flux statistics — the article's benchmark
  only guarantees agreement within a factor of three
- When 12CO cannot be assumed optically thick, since the semi-analytical flux scaling
  with disc surface area rests on that assumption
- When the target discs fall outside the surface-density/characteristic-radius grid
  validated against DALI in the Letter
tools:
- name: COpops
  role: Provides the disc parameter grids and the semi-analytical 12CO flux evaluation
    that is validated against DALI reference values
  repo: github.com/fzagaria/COpops
- name: DALI
  role: Supplies the reference thermochemical 12CO fluxes used as the benchmark standard
    for the factor-of-three agreement check
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:73f665fc7efb3573eaf7f819a85812e894b7d510a8a592f2d0adc40ec9dec08e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
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

# model-output-validation-against-reference-values

## Summary

Validate a semi-analytical 12CO disc-flux model by benchmarking its per-grid-point fluxes against reference values from DALI thermochemical models, accepting agreement within a factor of three.

## When to use

Use when a fast surrogate or semi-analytical model (e.g. COpops 12CO fluxes) must be checked against a reference set of expensive physical model outputs (e.g. DALI thermochemical fluxes) before population-scale use.

## When NOT to use

- When sub-factor-of-three precision is required, e.g. individual-disc physico-chemical characterisation rather than population-level flux statistics — the article's benchmark only guarantees agreement within a factor of three
- When 12CO cannot be assumed optically thick, since the semi-analytical flux scaling with disc surface area rests on that assumption
- When the target discs fall outside the surface-density/characteristic-radius grid validated against DALI in the Letter

## Inputs

- Representative disc parameter grid from COpops (power-law surface-density and characteristic-radius combinations)
- Reference 12CO fluxes from DALI thermochemical models for the same grid points
- Semi-analytically computed 12CO fluxes per grid point (optically thick assumption)

## Outputs

- co_flux_table.csv: per-grid-point 12CO fluxes keyed by disc parameters
- Per-point agreement ratios (semi-analytical vs DALI) documenting the within-factor-of-three benchmark

## How to apply

Evaluate the semi-analytical model over the representative disc parameter grid (power-law surface-density and characteristic-radius combinations shipped with COpops), computing 12CO fluxes under the optically thick assumption. For each grid point, look up the corresponding DALI thermochemical benchmark flux and compute the model/reference ratio. Apply the article's acceptance criterion: the semi-analytical fluxes must recover agreement within a factor of three of the DALI results. Assemble the validated fluxes into a single table (co_flux_table.csv) keyed by the disc parameters, so any point failing the factor-of-three check can be traced back to its surface-density/temperature profile parameters. The rationale is that the factor-of-three tolerance is explicitly established in the Letter as sufficient for population-level disc-evolution studies where running DALI for hundreds of discs is prohibitively expensive.

## Related tools

- **COpops** (Provides the disc parameter grids and the semi-analytical 12CO flux evaluation that is validated against DALI reference values) — github.com/fzagaria/COpops
- **DALI** (Supplies the reference thermochemical 12CO fluxes used as the benchmark standard for the factor-of-three agreement check)

## Evaluation signals

- Every computed 12CO flux agrees with its DALI reference value within a factor of three
- co_flux_table.csv contains one row per grid point, keyed by the disc surface-density and characteristic-radius parameters
- Fluxes are computed under the optically thick 12CO assumption and scale with disc surface area as the article states
- Grid coverage matches the representative disc parameter combinations studied in the Letter (Lupus and Upper Sco context)
- Any grid point deviating more than a factor of three from DALI is flagged and excluded from downstream population synthesis

## Limitations

- The benchmark tolerance is coarse: agreement is only within a factor of three, not percent-level, so the method is suited to population studies rather than individual-disc inference
- Uncertainties on initial conditions limited interpretation of the observations in the source work
- Available ALMA data have not yet given conclusive answers on the underlying disc evolution mechanisms (viscosity vs MHD winds), so validation against data beyond the DALI benchmark is inconclusive

## Evidence

- [abstract] Factor-of-three DALI benchmark (abstract): "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [abstract] Optically thick scaling basis: "assuming that 12CO is optically thick, fluxes scale with the disc surface area"
- [intro] Rationale for fast surrogate over expensive thermochemical runs: "Here we introduce an alternative method based on modelling semi-analytically the disc CO fluxes. This method is fast, flexible and accurate"
- [readme] README confirmation of factor-of-three agreement: "compared to more-complex physico-chemical models, we recover a good agreement within a factor of three"
