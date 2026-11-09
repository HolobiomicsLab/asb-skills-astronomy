---
name: strang-split-leapfrog-time-integration
description: Use when implementing or porting a Vlasov/hybrid-Vlasov solver that must
  advance a 6D distribution function with noiseless semi-Lagrangian propagation, split
  into translation and acceleration substeps.
when_to_use_negative:
- The target problem is a fully kinetic particle-in-cell or pure MHD system, not a
  Vlasov-equation distribution-function update
- Only a quick qualitative fluid estimate is needed — Vlasiator's semi-Lagrangian
  remapping exists precisely to avoid particle/MHD noise, at large computational cost
- The velocity-space rotation per step must exceed the ~22-degree limit and subcycling
  cannot be afforded
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Vlasiator
  role: Reference implementation of the Strang-split leapfrog hybrid-Vlasov solver;
    consult its source and SLICE-3D remapping library to ground the numerical design
    of the semi-Lagrangian propagation and field-solver modules
  repo: github.com/fmihpc/vlasiator
- name: analysator
  role: Python tools to read and analyze Vlasiator output files (VLSV) after the integration
    run
  repo: github.com/fmihpc/analysator
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:05048f1fe42bf1208bf686c653a9a498de858c27c7ff1b7260108de168bd60b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Vlasiator
  - analysator
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# strang-split-leapfrog-time-integration

## Summary

Strang-split leapfrog time integration for hybrid-Vlasov plasma solvers as implemented in Vlasiator: a full-step configuration-space translation combined with half-step velocity-space acceleration steps, using flux-conservative semi-Lagrangian remapping of the ion distribution function.

## When to use

Use when implementing or porting a Vlasov/hybrid-Vlasov solver that must advance a 6D distribution function with noiseless semi-Lagrangian propagation, split into translation and acceleration substeps.

## When NOT to use

- The target problem is a fully kinetic particle-in-cell or pure MHD system, not a Vlasov-equation distribution-function update
- Only a quick qualitative fluid estimate is needed — Vlasiator's semi-Lagrangian remapping exists precisely to avoid particle/MHD noise, at large computational cost
- The velocity-space rotation per step must exceed the ~22-degree limit and subcycling cannot be afforded

## Inputs

- Ion velocity distribution function on a 6D (configuration + velocity) grid
- Magnetic and electric field arrays updated by the constrained-transport field solver
- Solver configuration (grid extents, reduced 1D velocity space, timestep)
- Vlasiator source tree and SLICE-3D remapping library, cloned with --recurse-submodules

## Outputs

- Updated ion distribution function after translation and acceleration substeps
- Evolved field data under the Darwin/Hall-MHD closure
- End-of-run data file in VLSV, HDF5, or NumPy format produced by the driver script

## How to apply

Implement plasma propagation as a Strang-split leapfrog: apply a full-step configuration-space translation of the distribution function, sandwiched between half-step velocity-space acceleration steps, following the scheme described for Vlasiator. Use SLICE-3D-style flux-conservative semi-Lagrangian column remapping, with 5th-order reconstruction for the acceleration step and 3rd-order for the translation step, rearranging velocity space into single-cell columns parallel to the shear direction before each 1D remapping. When the velocity-space offset must be rotated via shear decomposition, enforce the ~22-degree per-step rotation limit; if a larger rotation angle is required, apply subcycling to split it into compliant substeps. Close the coupled system with an upwind constrained-transport field solver under the Darwin approximation and the Hall-MHD Ohm's-law relation, then validate end-to-end by running a small 1D test problem (reduced velocity space) and checking the driver completes and writes a parseable VLSV/HDF5/NumPy file.

## Related tools

- **Vlasiator** (Reference implementation of the Strang-split leapfrog hybrid-Vlasov solver; consult its source and SLICE-3D remapping library to ground the numerical design of the semi-Lagrangian propagation and field-solver modules) — github.com/fmihpc/vlasiator
- **analysator** (Python tools to read and analyze Vlasiator output files (VLSV) after the integration run) — github.com/fmihpc/analysator

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator && git submodule update --init --recursive
```

## Evaluation signals

- The driver script runs end-to-end without error (script_runs)
- The output file exists after the run (file_exists)
- The written data file parses as the declared VLSV, HDF5, or NumPy format (file_format_is)
- Velocity-space offset rotations per step never exceed the ~22-degree limit; larger angles trigger subcycling
- Remapping order matches the scheme: 5th-order reconstruction for acceleration, 3rd-order for translation

## Limitations

- Vlasiator's multi-dimensional approach at ion scales has immense computational challenges, requiring massively parallel computations on tens of thousands of cores
- The paper reports no quantitative threshold for the minimal 1D test; acceptance is only the qualitative check that the run completes and produces a parseable data file
- Dependent libraries are being transferred to git submodules; cloning without --recurse-submodules may leave the tree incomplete

## Evidence

- [results] SLICE-3D shear-column remapping: [evidence span withheld — anchor status: not_found]
- [readme] Vlasiator ion distribution functions: [evidence span withheld — anchor status: external_doc]
- [readme] numerically noiseless outcome: [evidence span withheld — anchor status: external_doc]
- [methods] Vlasiator build/install path: "https://github.com/fmihpc/vlasiator/wiki/Installing-Vlasiator"
