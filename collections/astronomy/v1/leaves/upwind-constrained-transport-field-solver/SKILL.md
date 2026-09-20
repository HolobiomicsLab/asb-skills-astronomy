---
name: upwind-constrained-transport-field-solver
description: Reference for implementing or modifying the electromagnetic field-update
  step of an ionosphere-free hybrid-Vlasov solver (e.g. Vlasiator-style), where ions
  are distribution functions, electrons are an MHD fluid, and a noiseless, divergence-constrained
  field evolution is required.
when_to_use_negative:
- Problems requiring full electromagnetic (light-wave/displacement-current) physics
  — the Darwin approximation removes them
- Fully kinetic systems where electrons must also be evolved as distribution functions
  rather than an MHD fluid
- Single-fluid ideal-MHD simulations with no ion distribution function to couple to
  the Hall-MHD Ohm's-law closure
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Vlasiator
  role: Reference implementation providing the upwind CT field solver under the Darwin
    approximation with Hall-MHD Ohm's-law closure in a global hybrid-Vlasov magnetospheric
    simulation
  repo: https://github.com/fmihpc/vlasiator
- name: analysator
  role: Python tools to read and analyze the VLSV output files produced by the coupled
    solver run
  repo: https://github.com/fmihpc/analysator
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:4132ddd1840dc11d629a13ed545a5d8ace03e236d097d2432140ba011fc3f84e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Vlasiator
  - analysator
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# upwind-constrained-transport-field-solver

## Summary

Method for advancing electromagnetic fields in a hybrid-Vlasov plasma simulation using an upwind constrained-transport (CT) field solver under the Darwin approximation, closed with the Hall-MHD Ohm's-law relation, as implemented in the Vlasiator global magnetospheric code.

## When to use

Reference for implementing or modifying the electromagnetic field-update step of an ionosphere-free hybrid-Vlasov solver (e.g. Vlasiator-style), where ions are distribution functions, electrons are an MHD fluid, and a noiseless, divergence-constrained field evolution is required.

## When NOT to use

- Problems requiring full electromagnetic (light-wave/displacement-current) physics — the Darwin approximation removes them
- Fully kinetic systems where electrons must also be evolved as distribution functions rather than an MHD fluid
- Single-fluid ideal-MHD simulations with no ion distribution function to couple to the Hall-MHD Ohm's-law closure

## Inputs

- Ion velocity distribution function (hybrid-Vlasov grid, e.g. Vlasiator velocity-space cells)
- Magnetic and electric field arrays on the configuration-space mesh (Vlasiator spatial grid)
- Plasma moments (density, velocity) computed from the distribution function for the Hall-MHD Ohm's-law closure

## Outputs

- Updated magnetic/electric field arrays on the simulation mesh after the CT update
- Coupled distribution-function and field data written per time step to VLSV (or HDF5/NumPy) files readable by Analysator

## How to apply

Consult the Vlasiator repository (github.com/fmihpc/vlasiator) and the Living Reviews in Computational Astrophysics review (10.1007/s41115-018-0003-2, Section 5) for the design rationale of the field-solver module. Implement the field update as an upwind constrained-transport scheme: evolve the magnetic field with a transport step that preserves divergence-free structure (CT), using upwind rather than centered fluxes so that shocks and steep gradients in the magnetosheath/foreshock context are handled stably. Operate under the Darwin approximation, i.e. neglect the displacement current so light-speed waves are removed while retaining self-consistent low-frequency electromagnetic coupling. Close the system with the Hall-MHD Ohm's-law relation, which is appropriate because in Vlasiator 'electrons are magnetohydrodynamic fluid' while ions are represented as velocity distribution functions, producing a self-consistent hybrid-Vlasov system. Couple the field solve to the Strang-split semi-Lagrangian Vlasov propagation (SLICE-3D remapping of the ion distribution) so fields and the distribution function are advanced consistently in the same driver loop.

## Related tools

- **Vlasiator** (Reference implementation providing the upwind CT field solver under the Darwin approximation with Hall-MHD Ohm's-law closure in a global hybrid-Vlasov magnetospheric simulation) — https://github.com/fmihpc/vlasiator
- **analysator** (Python tools to read and analyze the VLSV output files produced by the coupled solver run) — https://github.com/fmihpc/analysator

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator && git submodule update --init --recursive
```

## Evaluation signals

- The end-to-end driver script (coupling Vlasov propagation and CT field solve) runs without error (script_runs)
- The driver writes an output file and the file exists (file_exists)
- The written data file is in the declared VLSV/HDF5/NumPy format and is parseable (file_format_is), e.g. readable via analysator's VlsvReader
- Divergence-free structure of the magnetic field is maintained by the constrained-transport update across steps
- Velocity-space offset rotations respect the ~22-degree per-step limit, applying subcycling for larger angles so the field–plasma coupling remains stable

## Limitations

- The paper reports no quantitative threshold for a minimal 1D reduced-velocity-space test; acceptance is the qualitative check that the run completes and produces a parseable data file
- Computational challenges are immense — the multi-dimensional ion-scale approach relies on massively parallel HPC (tens of thousands of cores), so naive serial deployment will not scale to global magnetosphere runs
- When cloning the repository, git submodules must be initialized (--recurse-submodules, git submodule update --init --recursive) or dependent header libraries will be missing

## Evidence

- [methods] Field solver as upwind constrained-transport under Darwin approximation: [evidence span withheld — anchor status: not_found]
- [readme] Hybrid-Vlasov electron/ion split in Vlasiator: [evidence span withheld — anchor status: external_doc]
- [intro] Vlasiator as the prime global hybrid-Vlasov application: "The world’s (so far) only global magnetospheric hybrid-Vlasov simulation, the massively parallel Vlasiator, is therefore the prime application in this article."
- [readme] Submodule initialization requirement when cloning: [evidence span withheld — anchor status: external_doc]
