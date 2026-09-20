---
name: co-flux-semi-analytical-modelling
description: Use when comparing disc evolution models (viscous vs MHD-wind) against
  ALMA 12CO flux observations of disc populations (e.g. Lupus, Upper Sco) and synthesising
  population-level CO fluxes cheaply.
when_to_use_negative:
- When high-fidelity, disc-specific thermochemical outputs are required rather than
  fast population synthesis — DALI thermochemical models are the more complex alternative
  this method trades accuracy for speed against
- When the CO-emitting region cannot be treated as optically thick, since the method's
  flux scaling relies on fluxes scaling with disc surface area
edam_operation: http://edamontology.org/operation_3663
tools:
- name: COpops
  role: Repository implementing the semi-analytical 12CO flux model and evolutionary
    tracks used to predict disc CO fluxes
  repo: github.com/fzagaria/COpops
- name: DALI
  role: Thermochemical benchmark against which the semi-analytical model's agreement
    (within a factor of three) is validated
- name: ALMA
  role: Instrument providing the observed 12CO fluxes and disc sizes of the Lupus
    and Upper Sco disc samples used for proof-of-concept comparison
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:bbc7bee9a5aaa1f939b735e5bf47dd8d108765eb94e59178634fb9daa26951bc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA
  - COpops
  - DALI
  edam_operation: http://edamontology.org/operation_3663
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# co-flux-semi-analytical-modelling

## Summary

A fast, flexible, accurate semi-analytical method to compute 12CO fluxes for populations of protoplanetary discs, benchmarked against DALI thermochemical models within a factor of three, avoiding computationally-expensive thermochemical modelling of hundreds of discs.

## When to use

Use when comparing disc evolution models (viscous vs MHD-wind) against ALMA 12CO flux observations of disc populations (e.g. Lupus, Upper Sco) and synthesising population-level CO fluxes cheaply.

## When NOT to use

- When high-fidelity, disc-specific thermochemical outputs are required rather than fast population synthesis — DALI thermochemical models are the more complex alternative this method trades accuracy for speed against
- When the CO-emitting region cannot be treated as optically thick, since the method's flux scaling relies on fluxes scaling with disc surface area

## Inputs

- Survey catalogs of ALMA 12CO integrated line fluxes and disc sizes (e.g. Lupus 887-micron sizes, Upper Sco CO-disc radii)
- Viscous and MHD-wind disc evolutionary tracks parameterised in the COpops package (initial disc mass and radius ranges)
- CO-depletion prescription toggle (gas-phase CO abundance reduced from the canonical value)

## Outputs

- Predicted 12CO fluxes per disc or track, with and without CO depletion
- Flux-size comparison tracks overlaid on the observed flux-size plane
- Per-disc and sample-level model-vs-data comparison tables (e.g. upper_sco_model_comparison.csv, initial_condition_sensitivity.csv)
- Flux-size comparison figures (e.g. upper_sco_flux_size_comparison.png)

## How to apply

Use the COpops implementation (github.com/fzagaria/COpops) to compute semi-analytical 12CO fluxes for viscous and MHD-wind evolutionary tracks, assuming 12CO emission is optically thick so that fluxes scale with the disc surface area. Benchmark the semi-analytical predictions against DALI thermochemical models, accepting agreement within a factor of three. When interpreting Lupus or Upper Sco survey data, switch a CO-depletion prescription on and off: models and data agree well only when CO depletion is included. Run tracks under both initial conditions to bracket the uncertainty, then compare flux-selected versus size-selected samples (reported increases: factor 1.3 for Lupus, 3.6 for Upper Sco).

## Related tools

- **COpops** (Repository implementing the semi-analytical 12CO flux model and evolutionary tracks used to predict disc CO fluxes) — github.com/fzagaria/COpops
- **DALI** (Thermochemical benchmark against which the semi-analytical model's agreement (within a factor of three) is validated)
- **ALMA** (Instrument providing the observed 12CO fluxes and disc sizes of the Lupus and Upper Sco disc samples used for proof-of-concept comparison)

## Examples

```
git clone https://github.com/fzagaria/COpops
```

## Evaluation signals

- Semi-analytical predictions agree with DALI thermochemical model outputs within a factor of three
- Model tracks agree with Lupus and Upper Sco observed fluxes on the flux-size plane only in the CO-depletion case
- Flux-based usable sample exceeds the size-based sample by a factor of 1.3 for Lupus and 3.6 for Upper Sco
- Sensitivity grid covers the full initial-condition (mass and radius) ranges parameterised in the COpops package

## Limitations

- Uncertainties on the initial conditions (initial disc mass and radius ranges) limited the interpretation of the Lupus and Upper Sco observations
- The available ALMA data have not given conclusive answers on the underlying disc evolution mechanisms (viscosity vs MHD winds)
- Accuracy is benchmarked only to within a factor of three against more complex physico-chemical models

## Evidence

- [abstract] We developed a semi-analytical model to compute 12CO fluxes: "We developed a semi-analytical model to compute 12CO fluxes"
- [abstract] benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [abstract] assuming that 12CO is optically thick, fluxes scale with the disc surface area: "assuming that 12CO is optically thick, fluxes scale with the disc surface area"
- [abstract] Models and data agree well only if CO depletion is considered: "Models and data agree well only if CO depletion is considered"
- [readme] compared to more-complex physico-chemical models, we recover a good agreement within a factor of three: "compared to more-complex physico-chemical models, we recover a good agreement within a factor of three"
