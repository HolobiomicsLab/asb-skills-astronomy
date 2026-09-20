---
name: analytic-shock-solution-comparison
description: Use when a code-validation task requires comparing an SPH solution to
  a known analytic shock solution — specifically when FleCSPH's Noh shock test (2D
  or 3D) can be generated, evolved with hydro drivers, and checked against the analytic
  density plateau.
when_to_use_negative:
- The target problem is self-gravitating astrophysics (e.g. single or binary stars
  with Newtonian gravity via the newtonian_3d driver) where no analytic shock plateau
  exists to compare against.
- A 1D shock validation is required — the shipped Noh example is only provided in
  2D/3D; use the Sod shock tube examples in 1D/2D/3D instead.
- The runtime target is a task-based backend other than MPI (e.g. Legion or Kokkos),
  since the current FleCSPH version is limited to only the MPI backend.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: FleCSPH
  role: Provides the Noh shock test example (2D/3D), the noh_{2-3}d generators in
    app/id_generators/, and the hydro_{2-3}d drivers used to evolve the particle data
  repo: github.com/laristra/flecsph
- name: FleCSI
  role: Compile-time configurable framework supplying the parallel octree data structures
    on which FleCSPH particle affinity and evolution are built
  repo: github.com/laristra/flecsi
- name: H5part
  role: File format used for particle data input/output; snapshot files written by
    the hydro drivers are read to extract density profiles
- name: MPI
  role: Required backend for building FleCSPH before running the hydro drivers; the
    only supported backend in the current version
- name: Spack
  role: Package manager used to install FleCSPH and its dependencies and to load the
    generator and driver executables
  repo: github.com/spack/spack
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:1636a829c177d3d8d303017a9854239ab473823dede3b4e47a9dd15e6c7139c8
schema_version: 0.2.0
metadata:
  license_tier: open
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
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# analytic-shock-solution-comparison

## Summary

A validation skill for SPH hydrodynamics codes: run FleCSPH's shipped 2D/3D Noh shock example, evolve the particle lattice with the hydro drivers, and quantify the SPH density field against the analytic Noh solution across resolutions.

## When to use

Use when a code-validation task requires comparing an SPH solution to a known analytic shock solution — specifically when FleCSPH's Noh shock test (2D or 3D) can be generated, evolved with hydro drivers, and checked against the analytic density plateau.

## When NOT to use

- The target problem is self-gravitating astrophysics (e.g. single or binary stars with Newtonian gravity via the newtonian_3d driver) where no analytic shock plateau exists to compare against.
- A 1D shock validation is required — the shipped Noh example is only provided in 2D/3D; use the Sod shock tube examples in 1D/2D/3D instead.
- The runtime target is a task-based backend other than MPI (e.g. Legion or Kokkos), since the current FleCSPH version is limited to only the MPI backend.

## Inputs

- Noh parameter file (.par) copied from the flecsph data/ subdirectory, edited for particle count and output cadence
- Initial lattice particle data in H5part format produced by noh_2d_generator / noh_3d_generator from app/id_generators/
- Analytic Noh solution reference (gamma=5/3, post-shock density plateau of 64x initial density, self-similar shock front)

## Outputs

- H5part snapshot files evolved by the hydro_2d / hydro_3d drivers
- Radial density profile and shock-front position at the final time
- noh_convergence_table.csv containing density-error (L1/L2) and convergence-rate entries for both 2D and 3D runs

## How to apply

Copy a Noh parameter file from the repository's data/ subdirectory and edit it to set the particle count and output cadence for each target resolution. Run noh_2d_generator and noh_3d_generator from app/id_generators/ to produce initial lattice particle data in H5part format, then evolve each setup with the hydro_2d and hydro_3d drivers (hydro evolution without gravity), built with the MPI backend since FleCSPH currently supports only MPI. Read the H5part snapshot files and extract the radial density profile and the shock-front position at the final time. Compute the L1/L2 density error against the analytic Noh solution — a uniform post-shock density plateau of 64 times the initial density for gamma=5/3 and a self-similarly expanding shock front. Repeat across particle resolutions, tabulate density errors and the observed convergence rate in noh_convergence_table.csv, and confirm errors decrease monotonically with increasing resolution.

## Related tools

- **FleCSPH** (Provides the Noh shock test example (2D/3D), the noh_{2-3}d generators in app/id_generators/, and the hydro_{2-3}d drivers used to evolve the particle data) — github.com/laristra/flecsph
- **FleCSI** (Compile-time configurable framework supplying the parallel octree data structures on which FleCSPH particle affinity and evolution are built) — github.com/laristra/flecsi
- **H5part** (File format used for particle data input/output; snapshot files written by the hydro drivers are read to extract density profiles)
- **MPI** (Required backend for building FleCSPH before running the hydro drivers; the only supported backend in the current version)
- **Spack** (Package manager used to install FleCSPH and its dependencies and to load the generator and driver executables) — github.com/spack/spack

## Examples

```
spack install flecsph && spack load flecsph
```

## Evaluation signals

- The post-shock density plateau in the SPH solution reaches 64 times the initial density, matching the analytic Noh solution for gamma=5/3.
- L1/L2 density errors decrease monotonically as particle resolution increases, toward the analytic Noh density plateau.
- noh_convergence_table.csv exists and contains density-error and convergence-rate entries for both the 2D and 3D runs.
- The observed shock-front position at the final time tracks the self-similar analytic shock expansion.
- Snapshot files are valid H5part and compatible with modern visualization software.

## Limitations

- The current version of FleCSPH only supports the MPI backend and does not take advantage of all FleCSI features.
- The Noh shock test is shipped only in 2D and 3D; no 1D Noh setup is provided among the example physics problems.
- Parameter files and initial data must be taken from the data/ subdirectory of the cloned (recursive) repository; README details on module support are truncated in the source.

## Evidence

- [readme] Shipped examples include Noh in 2D/3D: "- Sod shock tubes in 1D/2D/3D;
- Noh shock test in 2D/3D;
- Sedov blast waves 2D and 3D;"
- [readme] Parameter files and initial data live in data/: "Sample parameter files and the intial data can be found in the `data` subdirectory."
- [abstract] Particle I/O uses H5part format: "Data input/output uses the H5part format, compatible with modern visualization software."
- [abstract] Only MPI backend is supported: "the current version of FleCSPH does not take advantage of all features provided in FleCSI: it is limited to only having the support for the MPI backend."
- [intro] FleCSPH is an SPH solver for Lagrangian problems: "FleCSPH implements a smoothed-particle hydrodynamics (SPH) solver for the solution of Lagrangian problems in astrophysics and cosmology."
