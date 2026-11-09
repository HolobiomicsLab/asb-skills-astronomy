---
name: fast-multipole-gravity-solver-configuration
description: Use when a 3D self-gravitating particle simulation must be evolved with
  FMM gravity (linear/sublinear complexity instead of exact O(N^2) N-body) and momentum
  conservation must be validated.
when_to_use_negative:
- When exact pairwise gravitational forces are required — FMM is an approximation
  with linear (or even sublinear) complexity, unlike the exact N-body solver with
  quadratic complexity
- When angular momentum conservation must hold beyond the first order of the multipole
  expansion — exact conservation is only guaranteed at the implemented first order
- When a non-MPI runtime is required — the current FleCSPH version is limited to only
  having the support for the MPI backend and does not take advantage of all FleCSI
  features (e.g. Legion/Kokkos)
tools:
- name: FleCSPH
  role: SPH application providing FMM support for gravitational forces and example
    physics problems (Sod tubes, Noh, Sedov, single/binary stars with Newtonian gravity)
  repo: github.com/laristra/flecsph
- name: FleCSI
  role: Compile-time configurable framework supplying the parallel octree data structure
    that handles particle affinity and gravitation
  repo: github.com/laristra/flecsi
- name: MPI
  role: The only supported backend/runtime for executing the newtonian_3d driver and
    FMM gravity in parallel
- name: H5part
  role: Input/output format for particle snapshots from which initial/final momenta
    are computed
- name: Spack
  role: Package manager used to install FleCSPH and dependencies and load its generators
    and drivers (spack install flecsph; spack load flecsph)
  repo: github.com/spack/spack
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:e16a1f7bf1e3ad990b74eedc2821a0d814cf5ab07813757b57d297183af58beb
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
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# fast-multipole-gravity-solver-configuration

## Summary

Configure, build, and run FleCSPH's fast multipole method (FMM) for self-gravitating SPH problems on the FleCSI parallel octree, then verify linear and angular momentum conservation from H5part snapshots.

## When to use

Use when a 3D self-gravitating particle simulation must be evolved with FMM gravity (linear/sublinear complexity instead of exact O(N^2) N-body) and momentum conservation must be validated.

## When NOT to use

- When exact pairwise gravitational forces are required — FMM is an approximation with linear (or even sublinear) complexity, unlike the exact N-body solver with quadratic complexity
- When angular momentum conservation must hold beyond the first order of the multipole expansion — exact conservation is only guaranteed at the implemented first order
- When a non-MPI runtime is required — the current FleCSPH version is limited to only having the support for the MPI backend and does not take advantage of all FleCSI features (e.g. Legion/Kokkos)

## Inputs

- Pre-configured FleCSPH parameter file (e.g. copied from the repository's data/ subdirectory, such as sodtube_t1_n1000.par) edited with particle count and simulation end time
- H5part-format initial particle dataset produced by a FleCSPH generator (e.g. noh_3d_generator or sodtube_3d_generator)
- FleCSPH source tree cloned recursively from github.com/laristra/flecsph

## Outputs

- Final H5part particle snapshot evolved with FMM gravity under MPI
- Total linear momentum and total angular momentum values for initial and final states (e.g. written to momentum_conservation.csv)
- Installed FleCSPH executables: id generators (sodtube_{1-2-3}d_generator, sedov_{2-3}d_generator, ...) and drivers (hydro_{1-2-3}d, newtonian_3d)

## How to apply

Clone the FleCSPH repository recursively (git clone --recursive git@github.com:laristra/flecsph.git) and build with cmake flags such as -DENABLE_UNIT_TESTS=ON and -DLOG_STRIP_LEVEL=1, followed by make -j and make install so that the id generators and the newtonian_3d driver are produced; alternatively install via Spack (spack install flecsph; spack load flecsph). Copy a pre-configured parameter file from the repository's data/ subdirectory and edit it to set the particle count and simulation end time for a 3D self-gravitating problem (e.g. noh_3d_generator or sodtube_3d_generator inputs), then run the generator to emit the initial H5part particle dataset. Evolve the system with app/driver/newtonian_3d under MPI — FleCSPH currently supports only the MPI backend — using FMM gravity as implemented at the first order of the multipole expansion, where particle affinity and gravitation are handled by FleCSI's parallel octree. Finally, compute total linear momentum (sum of mass*velocity) and total angular momentum (sum of r x m*v) from the initial and final H5part snapshots; tune the MAC angle if FMM timing is poor, since timing strongly depends on it.

## Related tools

- **FleCSPH** (SPH application providing FMM support for gravitational forces and example physics problems (Sod tubes, Noh, Sedov, single/binary stars with Newtonian gravity)) — github.com/laristra/flecsph
- **FleCSI** (Compile-time configurable framework supplying the parallel octree data structure that handles particle affinity and gravitation) — github.com/laristra/flecsi
- **MPI** (The only supported backend/runtime for executing the newtonian_3d driver and FMM gravity in parallel)
- **H5part** (Input/output format for particle snapshots from which initial/final momenta are computed)
- **Spack** (Package manager used to install FleCSPH and dependencies and load its generators and drivers (spack install flecsph; spack load flecsph)) — github.com/spack/spack

## Examples

```
cmake .. -DENABLE_UNIT_TESTS=ON -DLOG_STRIP_LEVEL=1 && make -j && make install
```

## Evaluation signals

- Total linear momentum (sum of mass*velocity over all particles) changes by ~0 between the initial and final H5part snapshots — FMM conserves linear momentum due to the symmetry of nodal interactions
- Total angular momentum (sum of r x m*v) changes by ~0 — conserved exactly at the first order of the multipole expansion
- spack load flecsph exposes the expected generators (sodtube_{1-2-3}d_generator, sedov_{2-3}d_generator, ...) and drivers (hydro_{1-2-3}d, newtonian_3d)
- FMM timing behaves as expected when the MAC angle is varied — timing strongly depends on the MAC angle, so anomalous flat timing suggests misconfiguration
- Final particle state is written in H5part format compatible with modern visualization software

## Limitations

- The current version of FleCSPH only supports the MPI backend and does not take advantage of all FleCSI features
- FMM is approximate: unlike the exact N-body with quadratic complexity, it has linear (or even sublinear) complexity, and angular momentum conservation is only exact at the implemented first order of the multipole expansion
- The timing of the FMM algorithm strongly depends on the MAC angle, so performance tuning is parameter-sensitive

## Evidence

- [abstract] FMM conserves momentum at first order: "FMM conserves linear momentum. In the implemented first order of the multipole expansion, angular momentum is conserved exactly."
- [intro] FMM gravity support in FleCSPH: "FleCSPH includes support for gravitational forces using the fast multipole method (FMM)."
- [intro] Octree-based gravitation: "Currently, particle affinity and gravitation is handled using the parallel implementation of the octree data structure provided by FleCSI"
- [abstract] MAC angle controls timing: "The timing of FMM algorithm strongly depends on the MAC angle."
- [methods] Build and install steps: "cmake .. -DCMAKE_BUILD_TYPE=debug -DENABLE_UNIT_TESTS=ON"
- [readme] Spack install path: "spack install flecsph"
- [abstract] Only MPI backend: "the current version of FleCSPH does not take advantage of all features provided in FleCSI: it is limited to only having the support for the MPI backend."
