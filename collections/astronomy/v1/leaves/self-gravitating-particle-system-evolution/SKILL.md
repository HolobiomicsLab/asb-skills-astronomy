---
name: self-gravitating-particle-system-evolution
description: Use when a 3D self-gravitating particle problem (e.g. single/binary stars,
  Noh 3D) must be evolved under Newtonian gravity with SPH, with initial data generated
  and snapshots stored in H5part.
when_to_use_negative:
- The problem is a pure hydrodynamics test without gravity (e.g. 1D/2D Sod tube or
  Sedov blast) — use the hydro_{1-2-3}d drivers instead of newtonian_3d
- Higher-than-first-order multipole accuracy is required — FleCSPH implements only
  the first order of the multipole expansion, at which angular momentum is conserved
  exactly
- A non-MPI runtime backend (Legion, Kokkos) is required — the current FleCSPH version
  is limited to only the MPI backend
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: FleCSPH
  role: SPH application providing the 3D initial-data generators and the newtonian_3d
    driver used to evolve the self-gravitating system with FMM gravity
  repo: https://github.com/laristra/flecsph
- name: FleCSI
  role: Provides the parallel octree data structures used for particle affinity and
    gravitation in the FMM computation
  repo: https://github.com/laristra/flecsi
- name: MPI
  role: Runtime backend over which the newtonian_3d driver is launched for distributed
    evolution
- name: H5part
  role: I/O format for the initial dataset and evolved particle snapshots
- name: Spack
  role: 'Alternative installation path: spack install flecsph builds all dependencies
    and installs generators and drivers'
  repo: https://github.com/spack/spack
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:012ec1077d25df751cbecb2138a2f7d66828d014a8733163dfb019a36240a469
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# self-gravitating-particle-system-evolution

## Summary

Evolve a 3D self-gravitating SPH particle system with FleCSPH's Newtonian driver, computing gravitational forces with the fast multipole method (FMM) on FleCSI's parallel octree, starting from an initial dataset generated in H5part format.

## When to use

Use when a 3D self-gravitating particle problem (e.g. single/binary stars, Noh 3D) must be evolved under Newtonian gravity with SPH, with initial data generated and snapshots stored in H5part.

## When NOT to use

- The problem is a pure hydrodynamics test without gravity (e.g. 1D/2D Sod tube or Sedov blast) — use the hydro_{1-2-3}d drivers instead of newtonian_3d
- Higher-than-first-order multipole accuracy is required — FleCSPH implements only the first order of the multipole expansion, at which angular momentum is conserved exactly
- A non-MPI runtime backend (Legion, Kokkos) is required — the current FleCSPH version is limited to only the MPI backend

## Inputs

- FleCSPH parameter file (.par) edited for a 3D self-gravitating problem (particle count, simulation end time)
- Initial particle dataset in H5part format produced by a 3D generator (e.g. noh_3d_generator, sodtube_3d_generator)
- FleCSPH build with MPI backend

## Outputs

- Evolved H5part particle snapshots (initial and final states)
- Per-step FMM gravitational accelerations on the particle set
- Momentum diagnostics (initial/final linear and angular momentum) computable from the snapshots

## How to apply

Build FleCSPH from the cloned laristra/flecsph repository (or install via Spack with 'spack install flecsph; spack load flecsph') to obtain the initial-data generators and the newtonian_3d driver. Copy a pre-configured parameter file from the repository's data/ subdirectory (e.g. cp ../data/sodtube_t1_n1000.par) and edit it to set the particle count and simulation end time for the 3D self-gravitating problem. Run the matching 3D generator (e.g. noh_3d_generator or sodtube_3d_generator) on the parameter file to produce the initial particle dataset in H5part format. Then evolve the system with app/driver/newtonian_3d under MPI, which computes gravity via FMM on FleCSI's parallel octree at the implemented first order of the multipole expansion; gravitational dynamics can be validated post hoc by summing mass*velocity (linear momentum) and r x m*v (angular momentum) over initial and final H5part snapshots.

## Related tools

- **FleCSPH** (SPH application providing the 3D initial-data generators and the newtonian_3d driver used to evolve the self-gravitating system with FMM gravity) — https://github.com/laristra/flecsph
- **FleCSI** (Provides the parallel octree data structures used for particle affinity and gravitation in the FMM computation) — https://github.com/laristra/flecsi
- **MPI** (Runtime backend over which the newtonian_3d driver is launched for distributed evolution)
- **H5part** (I/O format for the initial dataset and evolved particle snapshots)
- **Spack** (Alternative installation path: spack install flecsph builds all dependencies and installs generators and drivers) — https://github.com/spack/spack

## Examples

```
mpirun -n 4 ./app/driver/newtonian_3d newtonian.par
```

## Evaluation signals

- Total linear momentum (sum of mass*velocity over all particles) is unchanged between initial and final H5part snapshots (FMM conserves linear momentum due to symmetric nodal interactions)
- Total angular momentum (sum of r x m*v) is conserved to ~0 at the implemented first order of the multipole expansion
- The 3D generator produces a valid initial H5part dataset with the requested particle count before the driver run
- newtonian_3d completes under MPI without affinity/orphan-particle errors from the FleCSI octree
- FMM timing behaves as expected given the MAC angle used in the parameter file

## Limitations

- Angular momentum conservation is only guaranteed at the implemented first order of the multipole expansion; higher-order corrections are not implemented
- FMM computation cost strongly depends on the MAC angle chosen in the parameter file
- The current FleCSPH version only supports the MPI backend and does not take advantage of all FleCSI features
- FMM has linear (or sublinear) complexity, so it is approximate rather than the exact quadratic-complexity N-body force

## Evidence

- [abstract] FMM conserves linear momentum at first order: "FMM conserves linear momentum. In the implemented first order of the multipole expansion, angular momentum is conserved exactly."
- [intro] gravitation via FMM on FleCSI octree: "FleCSPH includes support
for gravitational forces using the fast multipole method (FMM)."
- [intro] octree handles particle affinity and gravitation: "Currently, particle affinity and gravitation is handled using the parallel
implementation of the octree data structure provided by FleCSI."
- [methods] build and install FleCSPH: "cmake .. \
       -DCMAKE_BUILD_TYPE=debug \
       -DENABLE_UNIT_TESTS=ON"
- [methods] copy and edit parameter file: "cp ../data/sodtube_t1_n1000.par sodtube.par"
- [readme] sample parameter files in data subdirectory: "Sample parameter files and the intial data can be found in the `data` subdirectory."
- [readme] available generators and drivers: "You will then have access to the generators and the drivers: 
- sodtube\_{1-2-3}d\_generator, sedov\_{2-3}d\_generator...
- hydro\_{1-2-3}d, newtonian\_3d..."
