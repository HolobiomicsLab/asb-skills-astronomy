---
name: hybrid-vlasov-solver-implementation
description: Use when implementing or re-implementing a (hybrid-)Vlasov solver that
  must propagate an ion velocity distribution function noiselessly and couple it to
  electromagnetic fields, following Vlasiator's documented numerical design.
when_to_use_negative:
- The problem requires kinetic electron physics — Vlasiator's hybrid approach treats
  electrons as a magnetohydrodynamic fluid, only ions are distribution functions
- The goal is only to read, plot, or analyze existing Vlasiator VLSV outputs rather
  than implement a solver — use the Analysator Python tools instead
- A single-fluid MHD global model already suffices; the skill targets non-MHD processes
  that require modelling ions as velocity distribution functions
tools:
- name: Vlasiator
  role: Reference implementation of the global hybrid-Vlasov solver whose Strang-split
    semi-Lagrangian Vlasov core and CT field solver are re-implemented
  repo: https://github.com/fmihpc/vlasiator
- name: Analysator
  role: Python tools to read and validate the VLSV output files produced by the solver
    test run
  repo: https://github.com/fmihpc/analysator.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:49840e21f627a37227188b4af0bc6605d0a73e00ed7e8d136d4a8399a1b3d54e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Analysator
  - Vlasiator
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# hybrid-vlasov-solver-implementation

## Summary

Implement a global hybrid-Vlasov plasma solver following Vlasiator's design: a Strang-split semi-Lagrangian Vlasov solver for the ion distribution function coupled to an upwind constrained-transport field solver under the Darwin approximation, closed with Hall-MHD Ohm's law (ions as velocity distribution functions, electrons as MHD fluid).

## When to use

Use when implementing or re-implementing a (hybrid-)Vlasov solver that must propagate an ion velocity distribution function noiselessly and couple it to electromagnetic fields, following Vlasiator's documented numerical design.

## When NOT to use

- The problem requires kinetic electron physics — Vlasiator's hybrid approach treats electrons as a magnetohydrodynamic fluid, only ions are distribution functions
- The goal is only to read, plot, or analyze existing Vlasiator VLSV outputs rather than implement a solver — use the Analysator Python tools instead
- A single-fluid MHD global model already suffices; the skill targets non-MHD processes that require modelling ions as velocity distribution functions

## Inputs

- Vlasiator repository (github.com/fmihpc/vlasiator) including SLICE-3D remapping library and git submodules
- Simulation configuration for a small 1D test problem with reduced velocity space
- Physical parameters for the ionosphere-free hybrid-Vlasov system (ion distribution function, Hall-MHD Ohm's-law closure)

## Outputs

- Driver script implementing the Strang-split semi-Lagrangian Vlasov propagation and CT field solver
- Output data file in VLSV, HDF5, or NumPy format containing the ion distribution function and field data

## How to apply

Clone the Vlasiator repository with `git clone --recurse-submodules https://github.com/fmihpc/vlasiator` and consult the SLICE-3D remapping library to ground the numerical design. Implement plasma propagation as a Strang-split leapfrog: a full-step configuration-space translation combined with half-step velocity-space acceleration steps, using SLICE-3D-style flux-conservative semi-Lagrangian column remapping with 5th-order reconstruction for the acceleration step and 3rd-order for the translation step; the SLICE-3D update splits remapping into 1D passes, rearranging velocity space into single-cell columns parallel to each shear direction. Implement the velocity-space offset rotation with shear decomposition, enforcing the ~22-degree per-step rotation limit and applying subcycling whenever a larger rotation angle is required. Implement the field solver as an upwind constrained-transport scheme under the Darwin approximation, closed with the Hall-MHD Ohm's-law relation for the ionosphere-free hybrid-Vlasov system. Set up a small 1D test problem with reduced velocity space, run the coupled solver end-to-end with a driver script, and write distribution-function and field data to a VLSV, HDF5, or NumPy file; since no quantitative threshold is reported for this minimal test, acceptance is the qualitative check that the run completes and produces a parseable file.

## Related tools

- **Vlasiator** (Reference implementation of the global hybrid-Vlasov solver whose Strang-split semi-Lagrangian Vlasov core and CT field solver are re-implemented) — https://github.com/fmihpc/vlasiator
- **Analysator** (Python tools to read and validate the VLSV output files produced by the solver test run) — https://github.com/fmihpc/analysator.git

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator
```

## Evaluation signals

- script_runs: the end-to-end driver script executes without error
- file_exists: the output data file is produced by the run
- file_format_is: the written data file is in the declared VLSV, HDF5, or NumPy format
- The velocity-space offset rotation never exceeds the ~22-degree per-step limit; larger required angles trigger subcycling
- The coupled solver completes a full Strang cycle (half-step acceleration, full-step translation, half-step acceleration) on the reduced-velocity-space 1D test problem

## Limitations

- The paper reports no quantitative threshold for the minimal 1D test; acceptance is only the qualitative check that the run completes and produces a parseable data file
- Vlasiator's computational challenges are immense — the multi-dimensional approach at ion scales requires massively parallel HPC on tens of thousands of cores
- Dependent header libraries are migrating to git submodules, so cloning or checking out branches without `--recurse-submodules` will leave the build incomplete

## Evidence

- [abstract] A particular application of the Vlasov system is Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere.: "A particular application of the Vlasov system is Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere."
- [intro] Section 5 describes Vlasiator in detail and justifies the decisions made in the design of the code to aid those who would like to design their own (hybrid-)Vlasov system.: "Section 5 describes Vlasiator in detail and justifies the decisions made in the design of the code to aid those who would like to design their own (hybrid-)Vlasov system."
- [readme] In Vlasiator, ions are represented as velocity distribution functions, while electrons are magnetohydrodynamic fluid, enabling a self-consistent global plasma simulation: [evidence span withheld — anchor status: external_doc]
- [readme] git clone --recurse-submodules https://github.com/fmihpc/vlasiator: [evidence span withheld — anchor status: external_doc]
