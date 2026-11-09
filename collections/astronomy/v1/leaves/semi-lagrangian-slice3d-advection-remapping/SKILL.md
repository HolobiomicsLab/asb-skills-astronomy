---
name: semi-lagrangian-slice3d-advection-remapping
description: 'Use when implementing or re-implementing a hybrid-Vlasov solver''s propagation
  step: translating a 6D distribution function in configuration space and accelerating
  it in velocity space, requiring flux-conservative semi-Lagrangian remapping decomposed
  into 1D shear-column passes.'
when_to_use_negative:
- The problem requires full electromagnetic (light-wave-carrying) Maxwell fields —
  Vlasiator's field solver uses the Darwin approximation, so radiative EM dynamics
  are outside this skill's scope
- The target model treats electrons kinetically — this scheme represents electrons
  as MHD fluid and only ions as velocity distribution functions
- A fluid (MHD) closure suffices and resolving non-MHD, multi-temperature ion-scale
  processes via a distribution function is unnecessary
tools:
- name: Vlasiator
  role: Reference hybrid-Vlasov code whose SLICE-3D remapping library and Strang-split
    solver modules ground the numerical design of the semi-Lagrangian propagation
    step
  repo: github.com/fmihpc/vlasiator
- name: analysator
  role: Python tools to read and analyze the VLSV output files produced by the test
    run of the solver
  repo: https://github.com/fmihpc/analysator.git
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:7e055b0221fe62d1a579612638450ec7ec1a6528a2d9ce3d4a82c2cb2be34a36
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Vlasiator
  - analysator
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# semi-lagrangian-slice3d-advection-remapping

## Summary

Implementation of Vlasiator's semi-Lagrangian Vlasov propagation for the ion distribution function, in which SLICE-3D-style remapping is decomposed into 1D column passes with Strang-split translation and acceleration steps. This is the core numerics for noiseless hybrid-Vlasov plasma simulation, where ions are distribution functions rather than macroparticles.

## When to use

Use when implementing or re-implementing a hybrid-Vlasov solver's propagation step: translating a 6D distribution function in configuration space and accelerating it in velocity space, requiring flux-conservative semi-Lagrangian remapping decomposed into 1D shear-column passes.

## When NOT to use

- The problem requires full electromagnetic (light-wave-carrying) Maxwell fields — Vlasiator's field solver uses the Darwin approximation, so radiative EM dynamics are outside this skill's scope
- The target model treats electrons kinetically — this scheme represents electrons as MHD fluid and only ions as velocity distribution functions
- A fluid (MHD) closure suffices and resolving non-MHD, multi-temperature ion-scale processes via a distribution function is unnecessary

## Inputs

- Ion velocity distribution function (phase-space array on a spatial grid with reduced or full velocity space)
- Plasma bulk moments and electromagnetic field state (upwind constrained-transport / Darwin-approximation field data)
- Grid, velocity-space, and timestep parameters including the shear/rotation angle per step

## Outputs

- Updated ion distribution function after translation, acceleration, and offset-rotation substeps
- Simulation output written in VLSV, HDF5, or NumPy format from a driver-script test run

## How to apply

Split the plasma propagation into a Strang-split leapfrog: apply a full-step configuration-space translation sandwiched between two half-step velocity-space acceleration steps, remapping the distribution function with a SLICE-3D-style flux-conservative semi-Lagrangian scheme. Before each shear transformation, rearrange velocity space into single-cell columns parallel to the shear direction so each column needs only a one-dimensional remap. Use 5th-order reconstruction for the acceleration step and 3rd-order for the translation step. Rotate the velocity-space offset with shear decomposition, enforcing the ~22-degree per-step rotation limit; when a larger rotation angle is required, apply subcycling to split it into admissible increments. Couple the result to an upwind constrained-transport field solver under the Darwin approximation, closed with the Hall-MHD Ohm's-law relation for the ionosphere-free hybrid-Vlasov system.

## Related tools

- **Vlasiator** (Reference hybrid-Vlasov code whose SLICE-3D remapping library and Strang-split solver modules ground the numerical design of the semi-Lagrangian propagation step) — github.com/fmihpc/vlasiator
- **analysator** (Python tools to read and analyze the VLSV output files produced by the test run of the solver) — https://github.com/fmihpc/analysator.git

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator
```

## Evaluation signals

- The end-to-end driver script runs without error (script_runs check)
- The declared output file exists on disk after the run (file_exists check)
- The written data file parses in the declared VLSV, HDF5, or NumPy format (file_format_is check)
- Each velocity-space offset rotation step respects the ~22-degree limit or is handled by subcycling
- For the minimal 1D test problem with reduced velocity space, acceptance is qualitative — the coupled run completes and produces a parseable data file, since no quantitative threshold is reported

## Limitations

- Field solver operates under the Darwin approximation with constrained transport, so displacement-current-driven EM wave dynamics are not captured
- The ~22-degree per-step velocity-space rotation limit forces subcycling whenever a larger rotation angle is needed, adding cost to strongly accelerating flows
- No quantitative accuracy threshold is reported for the minimal test; validation rests on qualitative checks that the run completes and the output file is parseable

## Evidence

- [full_text] SLICE-3D shear-column decomposition: [evidence span withheld — anchor status: not_found]
- [readme] Ions as distribution functions, electrons as MHD fluid: [evidence span withheld — anchor status: external_doc]
- [intro] Design-rationale purpose of the review article: "Section 5 describes Vlasiator in detail and justifies the decisions made in the design of the code to aid those who would like to design their own (hybrid-)Vlasov system."
- [abstract] Vlasiator as the global hybrid-Vlasov application: "Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere."
