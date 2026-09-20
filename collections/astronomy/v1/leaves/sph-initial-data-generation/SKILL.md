---
name: sph-initial-data-generation
description: 'Use when a FleCSPH simulation run is blocked because no initial particle
  data exists: the task names a shipped example problem (Sod tube, Noh, Sedov, oscillating
  star) and requires generating the lattice/particle setup before invoking hydro_2d/3d
  or newtonian drivers.'
when_to_use_negative:
- When initial particle data already exists and the goal is to evolve it — use the
  hydro_2d/hydro_3d or newtonian drivers instead of a generator
- When the physics problem has no matching shipped generator in app/id_generators
  (the paper lists only Sod shock tubes, Noh, Sedov, airfoil, spherical implosion,
  and single/binary stars as provided examples)
- When you only need to read or visualize existing H5part snapshots rather than create
  new particle setups
tools:
- name: FleCSPH
  role: Provides the initial-data generator executables (sodtube_{1-2-3}d_generator,
    sedov_{2-3}d_generator, noh generators, oscillating star generator) that write
    initial particle data in H5part format
  repo: github.com/laristra/flecsph
- name: Spack
  role: Package manager used to install FleCSPH and load its generator and driver
    executables into the environment ('spack install flecsph', 'spack load flecsph')
  repo: github.com/spack/spack
- name: H5part
  role: I/O format in which the generators write the initial particle data, compatible
    with the downstream hydro/newtonian drivers and visualization software
- name: FleCSI
  role: Compile-time configurable framework underlying FleCSPH's parallel data structures,
    which the generators and drivers are built on
  repo: github.com/laristra/flecsi
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:dfa1df921f5aa2c9eb858b765f2a0c036dd91a7a44c311d76a82414d39d8d67e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FleCSI
  - FleCSPH
  - H5part
  - MPI
  - Spack
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# sph-initial-data-generation

## Summary

Generates initial particle distributions for SPH simulations with FleCSPH's id_generators executables (e.g. sodtube_{1-2-3}d_generator, noh_2d/3d_generator, sedov_{2-3}d_generator, oscillating star generators), driven by pre-configured .par parameter files from the repository's data/ subdirectory, producing H5part-formatted initial data ready for the hydro or newtonian evolution drivers.

## When to use

Use when a FleCSPH simulation run is blocked because no initial particle data exists: the task names a shipped example problem (Sod tube, Noh, Sedov, oscillating star) and requires generating the lattice/particle setup before invoking hydro_2d/3d or newtonian drivers.

## When NOT to use

- When initial particle data already exists and the goal is to evolve it — use the hydro_2d/hydro_3d or newtonian drivers instead of a generator
- When the physics problem has no matching shipped generator in app/id_generators (the paper lists only Sod shock tubes, Noh, Sedov, airfoil, spherical implosion, and single/binary stars as provided examples)
- When you only need to read or visualize existing H5part snapshots rather than create new particle setups

## Inputs

- Pre-configured FleCSPH .par parameter file copied from the repository's data/ subdirectory (e.g. sodtube_t1_n1000.par)
- Edited parameter file with target particle count and output cadence set
- Installed FleCSPH with generator executables available (via 'spack load flecsph')
- Recursively cloned FleCSPH repository providing app/id_generators and data/ subdirectory

## Outputs

- H5part-formatted initial particle data file for the chosen problem and dimensionality
- Working .par parameter file configured for the desired particle count and output cadence

## How to apply

Install FleCSPH with Spack ('spack install flecsph', then 'spack load flecsph') to expose the initial-data generators alongside the drivers, and clone the repository recursively ('git clone --recursive git@github.com:laristra/flecsph.git') to obtain the pre-configured parameter files and initial data in the data/ subdirectory. Select the generator matching the target problem and dimensionality — sodtube_{1-2-3}d_generator, noh_2d/3d_generator, sedov_{2-3}d_generator, or the oscillating star generator from app/id_generators — and copy the corresponding .par file (e.g. 'cp ../data/sodtube_t1_n1000.par sodtube.par') as the working parameter file. Edit the .par file to set the target particle count (for resolution sweeps) and output cadence before running the generator. The generator emits the initial particle distribution in H5part format, which is FleCSPH's data I/O format and is compatible with modern visualization software; that H5part file then becomes the input for the hydro_2d/hydro_3d (hydro without gravity) or newtonian (3D hydro with self-gravity via FMM) drivers.

## Related tools

- **FleCSPH** (Provides the initial-data generator executables (sodtube_{1-2-3}d_generator, sedov_{2-3}d_generator, noh generators, oscillating star generator) that write initial particle data in H5part format) — github.com/laristra/flecsph
- **Spack** (Package manager used to install FleCSPH and load its generator and driver executables into the environment ('spack install flecsph', 'spack load flecsph')) — github.com/spack/spack
- **H5part** (I/O format in which the generators write the initial particle data, compatible with the downstream hydro/newtonian drivers and visualization software)
- **FleCSI** (Compile-time configurable framework underlying FleCSPH's parallel data structures, which the generators and drivers are built on) — github.com/laristra/flecsi

## Examples

```
spack load flecsph; cp ../data/sodtube_t1_n1000.par sodtube.par && sodtube_1d_generator sodtube.par
```

## Evaluation signals

- 'spack load flecsph' exposes the expected generator executables (sodtube_{1-2-3}d_generator, sedov_{2-3}d_generator, noh_2d/3d_generator) alongside the drivers
- The generator runs against the edited .par file and produces an H5part file readable by the subsequent hydro/newtonian driver and visualization software
- The generated particle count matches the target number set in the copied .par parameter file
- The .par file was copied from the repository's data/ subdirectory (e.g. cp ../data/sodtube_t1_n1000.par sodtube.par) and edited rather than the shipped original being modified
- Downstream evolution (hydro_2d/hydro_3d or newtonian driver) starts successfully from the generated H5part initial data

## Limitations

- The current version of FleCSPH is limited to the MPI backend and does not take advantage of all FleCSI features, which constrains the runtime the generators/drivers can use
- Generators exist only for the example problems shipped with FleCSPH (Sod 1D/2D/3D, Noh 2D/3D, Sedov 2D/3D, airfoil, spherical implosion, single/binary stars); custom problems require new generator code
- The README's Spack module-support installation instruction is truncated mid-sentence, so the 'spack install lmod' step may need external documentation
- Initial data must be regenerated whenever the target particle count changes in the .par file

## Evidence

- [abstract] Generators produce H5part-format initial particle data: "Data input/output uses the H5part format, compatible with modern visualization software."
- [intro] FleCSPH ships example problems in 1D, 2D and 3D with associated generators: "We provide several examples of physics problems in 1D, 2D and 3D:

- Sod shock tubes in 1D/2D/3D;
- Noh shock test in 2D/3D;
- Sedov blast waves 2D and 3D;"
- [readme] Spack install exposes generators and drivers; parameter files live in data/: "You will then have access to the generators and the drivers: 
- sodtube\_{1-2-3}d\_generator, sedov\_{2-3}d\_generator..."
- [readme] Sample parameter files and initial data are in the data subdirectory: "Sample parameter files and the intial data can be found in the `data` subdirectory."
- [methods] Concrete copy-and-edit parameter file workflow step: "cp ../data/sodtube_t1_n1000.par sodtube.par"
- [methods] Recursive clone of the FleCSPH repository is required: "git clone --recursive git@github.com:laristra/flecsph.git"
