---
name: semi-analytical-disc-structure-modelling
description: Use when 12CO (J=2-1 or similar) line fluxes are needed for population-scale
  disc samples (hundreds of discs) and running thermochemical codes like DALI per
  disc would be prohibitively time-consuming.
when_to_use_negative:
- When high-fidelity disc chemistry or temperature structure is required per disc
  — the semi-analytical model only agrees with DALI within a factor of three, not
  line-by-line accuracy
- 'When the disc evolution mechanism (viscosity vs MHD winds) must be pinned down:
  the article states available ALMA data have not given conclusive answers yet, and
  uncertainties on initial conditions limited interpretation'
- When 12CO cannot be assumed optically thick — the flux–surface-area scaling central
  to the method relies on that assumption
tools:
- name: COpops
  role: Package implementing the semi-analytical disc structure and 12CO flux computation
    over representative disc parameter grids
  repo: github.com/fzagaria/COpops
- name: DALI
  role: Thermochemical reference model used only for benchmarking/validation of the
    semi-analytical fluxes
- name: ALMA
  role: Source of the observed 12CO disc samples (Lupus, Upper Sco) against which
    the modelled fluxes are compared
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:0b4f83d7c46b77b86b9db6e88c78d0148aac6db2155985748de80fdb324b9c28
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
yes_rate: 0.7
n_claims: 10
---

# semi-analytical-disc-structure-modelling

## Summary

Compute 12CO disc line fluxes with a fast, flexible semi-analytical model (power-law surface-density and temperature profiles) implemented in the COpops package, avoiding computationally-expensive thermochemical modelling of hundreds of discs while retaining accuracy within a factor of three of DALI benchmarks.

## When to use

Use when 12CO (J=2-1 or similar) line fluxes are needed for population-scale disc samples (hundreds of discs) and running thermochemical codes like DALI per disc would be prohibitively time-consuming.

## When NOT to use

- When high-fidelity disc chemistry or temperature structure is required per disc — the semi-analytical model only agrees with DALI within a factor of three, not line-by-line accuracy
- When the disc evolution mechanism (viscosity vs MHD winds) must be pinned down: the article states available ALMA data have not given conclusive answers yet, and uncertainties on initial conditions limited interpretation
- When 12CO cannot be assumed optically thick — the flux–surface-area scaling central to the method relies on that assumption

## Inputs

- Representative disc parameter grids from the COpops package (disc surface-density and characteristic-radius combinations)
- Disc structural parameters: power-law surface density and temperature profiles
- Optional comparison samples: ALMA 12CO flux measurements for Lupus and Upper Sco discs

## Outputs

- Per-grid-point semi-analytical 12CO line fluxes (e.g. assembled into co_flux_table.csv)
- Model-vs-data flux comparison for Lupus and Upper Sco disc populations

## How to apply

Load the representative disc parameter grids shipped with COpops (github.com/fzagaria/COpops), covering the disc surface-density and characteristic-radius combinations studied in the Letter. For each grid point, evaluate the semi-analytical disc structure (power-law surface density and temperature profiles) under the assumption that 12CO is optically thick, so the emergent flux scales with the disc surface area; integrate over the disc surface to obtain the line flux. Assemble per-grid-point fluxes into a table keyed by disc parameters (e.g. co_flux_table.csv). Validate against DALI thermochemical benchmarks, expecting agreement within a factor of three, and interpret population-level agreement with Lupus/Upper Sco ALMA data only when CO depletion is considered.

## Related tools

- **COpops** (Package implementing the semi-analytical disc structure and 12CO flux computation over representative disc parameter grids) — github.com/fzagaria/COpops
- **DALI** (Thermochemical reference model used only for benchmarking/validation of the semi-analytical fluxes)
- **ALMA** (Source of the observed 12CO disc samples (Lupus, Upper Sco) against which the modelled fluxes are compared)

## Evaluation signals

- Computed 12CO fluxes agree with DALI thermochemical benchmark fluxes within a factor of three
- Per-grid-point flux table is keyed by disc parameters (surface-density normalization, characteristic radius) with no missing grid points
- Under the optically thick assumption, fluxes scale consistently with the disc surface area across the parameter grid
- Population-level comparison with Lupus/Upper Sco data agrees only when CO depletion is applied — disagreement without CO depletion flags a missing physical ingredient
- Fluxes for a given grid point are reproducible from the COpops power-law surface-density and temperature profiles

## Limitations

- Agreement with full thermochemical models is only within a factor of three, insufficient for precision flux predictions
- Models and Lupus/Upper Sco data agree well only if CO depletion is considered; neglecting it biases fluxes
- Uncertainties on the initial conditions limited the interpretation of the observations
- Available ALMA data have not given conclusive answers yet on the underlying disc evolution mechanisms (viscosity vs MHD winds)

## Evidence

- [abstract] Semi-analytical model agrees with DALI within factor of three: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [abstract] Optically thick flux–area scaling: "assuming that 12CO is optically thick, fluxes scale with the disc surface area"
- [intro] Motivation: avoid expensive thermochemical models for hundreds of discs: "Here we introduce an alternative method based on modelling semi-analytically the disc CO fluxes. This method is fast, flexible and accurate"
- [abstract] CO depletion required for agreement with data: "Models and data agree well only if CO depletion is considered"
- [readme] README restates method speed and accuracy rationale: "running computationally-expensive thermochemical models for hundreds of discs can be prohibitively time-consuming"
