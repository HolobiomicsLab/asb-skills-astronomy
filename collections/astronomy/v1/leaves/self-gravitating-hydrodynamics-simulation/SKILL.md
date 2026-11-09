---
name: self-gravitating-hydrodynamics-simulation
description: Use when the task requires evolving 3D hydrodynamic systems with Newtonian
  self-gravity (e.g., oscillating stars, single/binary stars) via SPH and need reproducible
  particle evolutions across particle counts.
when_to_use_negative:
- Problem lacks self-gravity and is covered by pure-hydro examples (Sod shock tube
  1D/2D/3D, Noh 2D/3D, Sedov 2D/3D, airfoil) — use the hydro_{1-2-3}d drivers instead
  of newtonian
- A non-MPI or task-based runtime (Legion, Kokkos) is required — the current FleCSPH
  version is limited to only the MPI backend
- Exact N-body gravitational forces are required — FMM's first-order multipole expansion
  conserves linear momentum but exact angular momentum only at first order
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: FleCSPH
  role: SPH simulation application providing id_generators and the newtonian evolution
    driver for self-gravitating hydrodynamics
  repo: github.com/laristra/flecsph
- name: FleCSI
  role: Compile-time configurable framework supplying the parallel octree data structures
    used for particle affinity and gravitation
  repo: github.com/laristra/flecsi
- name: Spack
  role: Package manager used to install FleCSPH and its dependencies and load the
    executables into the environment
  repo: github.com/spack/spack
- name: H5part
  role: I/O format for simulation output, compatible with modern visualization software
- name: MPI
  role: The only backend supported by the current FleCSPH version for parallel execution
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:beaa74db529fd9038f0fd7c092bc91bc99927d18ed29e48d1bd78d151e5dcad2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FleCSI
  - FleCSPH
  - H5part
  - MPI
  - Spack
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# self-gravitating-hydrodynamics-simulation

## Summary

Run SPH simulations of self-gravitating astrophysical systems (e.g., oscillating stars) with FleCSPH, using FleCSI's parallel octree and FMM for gravity, from initial-data generation through H5part output of evolution diagnostics.

## When to use

Use when the task requires evolving 3D hydrodynamic systems with Newtonian self-gravity (e.g., oscillating stars, single/binary stars) via SPH and need reproducible particle evolutions across particle counts.

## When NOT to use

- Problem lacks self-gravity and is covered by pure-hydro examples (Sod shock tube 1D/2D/3D, Noh 2D/3D, Sedov 2D/3D, airfoil) — use the hydro_{1-2-3}d drivers instead of newtonian
- A non-MPI or task-based runtime (Legion, Kokkos) is required — the current FleCSPH version is limited to only the MPI backend
- Exact N-body gravitational forces are required — FMM's first-order multipole expansion conserves linear momentum but exact angular momentum only at first order

## Inputs

- Oscillating star (or other) .par parameter file from the flecsph 'data' subdirectory
- Target number of particles to simulate
- Generated initial particle distribution from an app/id_generators generator

## Outputs

- Evolved particle state files in H5part format
- Oscillation diagnostics (velocity and density) per particle count, e.g. oscillating_star_diagnostics.csv with one row per tested particle count

## How to apply

Install FleCSPH with Spack ('spack install flecsph' followed by 'spack load flecsph') to expose the initial-data generators and the hydro/newtonian drivers. Clone the FleCSPH repository recursively and locate pre-configured parameter files and initial data in the 'data' subdirectory. Copy the oscillating star .par file, edit the target particle count, and run the matching oscillating star generator from app/id_generators to produce the initial particle distribution. Evolve with the 'newtonian' driver (3D SPH hydro with self-gravity via FMM on FleCSI's parallel octree). Extract velocity and density oscillation diagnostics from the H5part output for each particle count and compile them into a CSV (one row per particle count) to check convergence against the paper's reported trends.

## Related tools

- **FleCSPH** (SPH simulation application providing id_generators and the newtonian evolution driver for self-gravitating hydrodynamics) — github.com/laristra/flecsph
- **FleCSI** (Compile-time configurable framework supplying the parallel octree data structures used for particle affinity and gravitation) — github.com/laristra/flecsi
- **Spack** (Package manager used to install FleCSPH and its dependencies and load the executables into the environment) — github.com/spack/spack
- **H5part** (I/O format for simulation output, compatible with modern visualization software)
- **MPI** (The only backend supported by the current FleCSPH version for parallel execution)

## Examples

```
spack install flecsph && spack load flecsph && git clone --recursive git@github.com:laristra/flecsph.git && cd flecsph && cp data/oscillating_star.par my_star.par && newtonian_3d my_star.par
```

## Evaluation signals

- oscillating_star_diagnostics.csv exists and contains one row per tested particle count
- Velocity and density oscillation diagnostics show behavior consistent with the paper's reported convergence trends across particle counts
- Simulation output is written in valid H5part format readable by modern visualization software
- Initial-data generator and newtonian driver executables are available after 'spack load flecsph'
- The evolved star remains a coherent oscillating configuration rather than dispersing, indicating gravity via FMM was applied correctly

## Limitations

- The current version of FleCSPH only supports the MPI backend and does not take advantage of all FleCSI features
- FMM timing strongly depends on the MAC angle, affecting performance tuning
- FMM conserves linear momentum, but angular momentum is conserved exactly only at first order of the multipole expansion; exact N-body would be quadratic cost

## Evidence

- [abstract] FMM gravity driver: "The asynchronous distributed tree topology combined with a fast multipole method allows FleCSPH to efficiently compute hydrodynamics and long range particle–particle interactions."
- [abstract] MPI-only limitation: "the current version of FleCSPH does not take advantage of all features provided in FleCSI: it is limited to only having the support for the MPI backend."
- [readme] Spack install/load workflow: "spack install flecsph"
- [readme] available drivers: "You will then have access to the generators and the drivers: 
- sodtube_{1-2-3}d_generator, sedov_{2-3}d_generator...
- hydro_{1-2-3}d, newtonian_3d..."
- [readme] parameter files location: "Sample parameter files and the intial data can be found in the `data` subdirectory."
- [abstract] H5part output format: "Data input/output uses the H5part format, compatible with modern visualization software."
