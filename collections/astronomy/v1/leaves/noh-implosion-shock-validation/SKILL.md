---
name: noh-implosion-shock-validation
description: Use when a new SPH build or configuration of FleCSPH (or a similar tree-based
  SPH code) needs numerical validation against a strong-shock benchmark with a known
  analytic density plateau and self-similar shock front.
when_to_use_negative:
- The goal is to test gravity/FMM coupling — the Noh setup is pure hydrodynamics evolved
  with the hydro_2d/hydro_3d drivers without gravity; use a self-gravitating example
  (e.g. Newtonian single/binary star problems) instead
- The code under test does not support MPI — FleCSPH's current version is limited
  to the MPI backend, so validation runs cannot be executed
- Validation of a problem class with a different analytic benchmark is required (e.g.
  Sod shock tubes, Sedov blast waves, airfoil flow) — use the corresponding shipped
  example instead
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: FleCSPH
  role: SPH application providing the shipped Noh shock test examples, parameter files,
    generators, and hydro drivers
  repo: https://github.com/laristra/flecsph
- name: FleCSI
  role: Compile-time configurable framework providing the parallel octree data structures
    on which FleCSPH runs
  repo: https://github.com/laristra/flecsi
- name: H5part
  role: I/O format for the generated initial particle data and the evolved snapshot
    files read for profile extraction
- name: MPI
  role: The only supported backend in the current FleCSPH version; required to build
    and run the hydro_2d/hydro_3d drivers
- name: Spack
  role: Package manager used to install FleCSPH and its dependencies and load the
    generator/driver executables
  repo: https://github.com/spack/spack
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:97af3927a9e221634686cf9342b79a7c61927bd14fe93ab7c6f2889051a8364b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/laristra/flecsph
  tools:
  - FleCSI
  - FleCSPH
  - H5part
  - MPI
  - Spack
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# noh-implosion-shock-validation

## Summary

Validate a smoothed-particle hydrodynamics (SPH) code by running FleCSPH's shipped Noh shock test example in 2D and 3D and comparing the simulated radial density profile against the analytic Noh solution.

## When to use

Use when a new SPH build or configuration of FleCSPH (or a similar tree-based SPH code) needs numerical validation against a strong-shock benchmark with a known analytic density plateau and self-similar shock front.

## When NOT to use

- The goal is to test gravity/FMM coupling — the Noh setup is pure hydrodynamics evolved with the hydro_2d/hydro_3d drivers without gravity; use a self-gravitating example (e.g. Newtonian single/binary star problems) instead
- The code under test does not support MPI — FleCSPH's current version is limited to the MPI backend, so validation runs cannot be executed
- Validation of a problem class with a different analytic benchmark is required (e.g. Sod shock tubes, Sedov blast waves, airfoil flow) — use the corresponding shipped example instead

## Inputs

- Noh shock parameter files from the FleCSPH data/ subdirectory (.par) with edited particle count and output cadence
- Initial Noh lattice particle data in H5part format produced by noh_2d_generator / noh_3d_generator
- Analytic Noh solution parameters (gamma = 5/3, post-shock density = 64 x initial density, self-similar shock-front trajectory)

## Outputs

- H5part snapshot files with evolved density and particle positions for each 2D/3D resolution
- Radial density profile and shock-front position extracted at the final time
- noh_convergence_table.csv with L1/L2 density errors and convergence rates for 2D and 3D runs

## How to apply

Install FleCSPH (e.g. via Spack with the MPI backend, since the current version is limited to MPI support), then copy a Noh parameter file from the repository's data/ subdirectory and edit it to set the particle count and output cadence for each target resolution. Generate the initial lattice particle data in H5part format using the noh_2d_generator and noh_3d_generator from app/id_generators/. Evolve each setup with the hydro_2d and hydro_3d drivers from app/drivers/ (pure hydro evolution, no gravity). Read the H5part snapshots, extract the radial density profile and shock-front position at the final time, and compute L1/L2 density errors against the analytic Noh solution, which for gamma=5/3 yields a uniform post-shock density plateau of 64 times the initial density. Tabulate errors and observed convergence rates across resolutions; the Noh test is a standard validation because its exact self-similar solution makes density errors directly attributable to the SPH discretization.

## Related tools

- **FleCSPH** (SPH application providing the shipped Noh shock test examples, parameter files, generators, and hydro drivers) — https://github.com/laristra/flecsph
- **FleCSI** (Compile-time configurable framework providing the parallel octree data structures on which FleCSPH runs) — https://github.com/laristra/flecsi
- **H5part** (I/O format for the generated initial particle data and the evolved snapshot files read for profile extraction)
- **MPI** (The only supported backend in the current FleCSPH version; required to build and run the hydro_2d/hydro_3d drivers)
- **Spack** (Package manager used to install FleCSPH and its dependencies and load the generator/driver executables) — https://github.com/spack/spack

## Examples

```
spack install flecsph; spack load flecsph
```

## Evaluation signals

- noh_convergence_table.csv exists and contains density-error and convergence-rate entries for both the 2D and 3D runs
- Computed L1/L2 density errors decrease monotonically with increasing particle resolution
- Post-shock density plateau approaches 64 times the initial density for gamma=5/3 as resolution increases
- Observed shock-front position tracks the analytic self-similar Noh trajectory at the final time
- Runs complete with the MPI-backed hydro_2d/hydro_3d drivers and produce readable H5part snapshots at the configured output cadence

## Limitations

- The current version of FleCSPH is limited to the MPI backend and does not take advantage of all FleCSI features, constraining the runtimes on which validation can be performed
- The Noh validation covers pure hydrodynamics only; it exercises none of the FMM gravitational-force capabilities
- Available pre-configured parameter files and initial data are limited to what ships in the data subdirectory of the laristra/flecsph repository

## Evidence

- [intro] Noh test shipped in 2D/3D: "We provide several examples of physics problems in 1D, 2D and 3D:

- Sod shock tubes in 1D/2D/3D;
- Noh shock test in 2D/3D;"
- [abstract] H5part output format: "Data input/output uses the H5part format, compatible with modern visualization software."
- [abstract] MPI-only backend: "the current version of FleCSPH does not take advantage of all features provided in FleCSI: it is limited to only having the support for the MPI backend."
- [readme] Parameter files and initial data location: "Sample parameter files and the intial data can be found in the `data` subdirectory."
- [readme] Spack installation and loading of executables: "spack install flecsph 
``nThis will build all the dependencies, compile and install FleCSPH. 
In order to use FleCSPH executables simply run: 
{engine=sh}
spack load flecsph"
