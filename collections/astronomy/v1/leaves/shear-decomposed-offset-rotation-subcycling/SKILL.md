---
name: shear-decomposed-offset-rotation-subcycling
description: Use when implementing or porting a semi-Lagrangian Vlasov solver in which
  the velocity mesh must be rotated/offset relative to the configuration-space grid,
  and the rotation angle per Strang-split step may exceed the stability-preserving
  limit.
when_to_use_negative:
- If your solver uses a fixed, non-offset velocity grid where no rotation between
  steps is needed (e.g. an Eulerian grid-aligned Vlasov discretization)
- If you are running a pure MHD or electron-fluid model with no distribution function
  to remap — Vlasiator models ions as distribution functions while electrons are MHD
  fluid
edam_operation: http://edamontology.org/operation_3891
tools:
- name: Vlasiator
  role: Reference hybrid-Vlasov implementation containing the SLICE-3D remapping library
    and the shear-decomposed offset rotation with the ~22-degree rotation limit and
    subcycling
  repo: https://github.com/fmihpc/vlasiator
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:524d32952aada5c4f1709de0eccad5ba275d8629bb4bb17de5b019580aeed746
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Vlasiator
  edam_operation: http://edamontology.org/operation_3891
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# Shear-decomposed offset rotation with subcycling

## Summary

A technique from Vlasiator's semi-Lagrangian Vlasov solver for rotating the velocity-space grid offset between acceleration steps: each rotation is decomposed into shears so the SLICE-3D remapping reduces to 1D passes, with the rotation limited to ~22 degrees per step and subcycling applied for larger angles.

## When to use

Use when implementing or porting a semi-Lagrangian Vlasov solver in which the velocity mesh must be rotated/offset relative to the configuration-space grid, and the rotation angle per Strang-split step may exceed the stability-preserving limit.

## When NOT to use

- If your solver uses a fixed, non-offset velocity grid where no rotation between steps is needed (e.g. an Eulerian grid-aligned Vlasov discretization)
- If you are running a pure MHD or electron-fluid model with no distribution function to remap — Vlasiator models ions as distribution functions while electrons are MHD fluid

## Inputs

- Ion velocity distribution function (6D phase-space grid, e.g. stored as VLSV/HDF5/NumPy arrays)
- Required velocity-space offset rotation angle per Strang-split step
- Shear direction (axis parallel to which velocity-space columns are formed)

## Outputs

- Remapped distribution function on the rotated velocity mesh, produced by 1D SLICE-3D column remappings
- Subcycling schedule (number of sub-steps) when the rotation angle exceeds the ~22-degree per-step limit

## How to apply

Within a Strang-split leapfrog (full-step configuration-space translation, half-step velocity-space acceleration), the relative offset between the velocity grid and real-space grid must be rotated after propagation. Rearrange velocity space into single-cell columns parallel to the shear direction before each shear transformation so each pass requires only a one-dimensional SLICE-3D remapping; use 5th-order reconstruction for acceleration passes and 3rd-order for translation passes, consistent with the paper's design. Enforce the ~22-degree maximum rotation per step: if the required rotation angle exceeds this limit, split the rotation into subcycles each within the limit rather than rotating in one operation. This preserves the accuracy and stability of the flux-conservative semi-Lagrangian column remapping while keeping every remapping one-dimensional.

## Related tools

- **Vlasiator** (Reference hybrid-Vlasov implementation containing the SLICE-3D remapping library and the shear-decomposed offset rotation with the ~22-degree rotation limit and subcycling) — https://github.com/fmihpc/vlasiator

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator
```

## Evaluation signals

- Rotation angle per applied step never exceeds the ~22-degree limit; larger requests produce multiple subcycles
- Each remapping pass is one-dimensional: velocity space is rearranged into single-cell columns parallel to the shear direction before each shear transformation
- Distribution function remains normalized/conserved after the rotation (flux-conservative semi-Lagrangian remapping)
- Coupled driver with reduced velocity space completes end-to-end and writes a parseable VLSV, HDF5, or NumPy output file (script_runs, file_exists, file_format_is)
- Reconstruction order matches the design: 5th-order for velocity-space acceleration steps, 3rd-order for configuration-space translation steps

## Limitations

- The rotation is limited to ~22 degrees per step; larger angles require subcycling, which increases computational cost of the remapping
- Rotation accuracy depends on the 1D column-remapping reconstruction orders (5th/3rd), so lower-order substitutions will degrade the semi-Lagrangian update
- No quantitative error threshold is given for the minimal coupled test; acceptance is the qualitative check that the run completes and produces a parseable data file

## Evidence

- [methods] SLICE-3D column decomposition: [evidence span withheld — anchor status: not_found]
- [methods] Rotation limit and subcycling: [evidence span withheld — anchor status: not_found]
- [methods] Strang-split propagation context: [evidence span withheld — anchor status: not_found]
- [readme] Ions as distribution functions: [evidence span withheld — anchor status: external_doc]
