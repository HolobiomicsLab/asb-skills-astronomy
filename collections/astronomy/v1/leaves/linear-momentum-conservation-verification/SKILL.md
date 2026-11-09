---
name: linear-momentum-conservation-verification
description: Use when validating an FMM-based gravitational solver for conservation
  properties — e.g., after evolving a 3D self-gravitating SPH problem (Noh, Sod tube,
  single/binary star) with FleCSPH's newtonian_3d driver and you have initial and
  final H5part snapshots to compare.
when_to_use_negative:
- The verification target is FMM performance or timing sensitivity to the MAC angle
  rather than momentum conservation properties
- The evolved system includes non-gravitational, non-momentum-conserving physics (e.g.
  wind-tunnel airfoil flow with boundary forces), where zero momentum change is not
  expected
- Only a single snapshot is available with no initial/final pair to difference
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: FleCSPH
  role: Provides the FMM gravitational solver, the 3D initial-data generators, and
    the newtonian_3d driver used to produce the snapshots being checked
  repo: github.com/laristra/flecsph
- name: FleCSI
  role: Supplies the parallel octree data structure over which symmetric nodal interactions
    give linear-momentum conservation
  repo: github.com/laristra/flecsi
- name: H5part
  role: File format of the initial and final particle snapshots from which momentum
    sums are computed
- name: MPI
  role: Backend used to run newtonian_3d for the distributed FMM evolution
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:5f6f3cfcee5e8dceed6130d7dd51ce26f40b34bcb9b158645de76b53de730b6b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FleCSI
  - FleCSPH
  - H5part
  - MPI
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# linear-momentum-conservation-verification

## Summary

Verifies that FleCSPH's fast multipole method (FMM) gravity conserves total linear momentum (and angular momentum at the first order of the multipole expansion) by comparing momentum sums computed from initial and final H5part snapshots of a self-gravitating simulation.

## When to use

Use when validating an FMM-based gravitational solver for conservation properties — e.g., after evolving a 3D self-gravitating SPH problem (Noh, Sod tube, single/binary star) with FleCSPH's newtonian_3d driver and you have initial and final H5part snapshots to compare.

## When NOT to use

- The verification target is FMM performance or timing sensitivity to the MAC angle rather than momentum conservation properties
- The evolved system includes non-gravitational, non-momentum-conserving physics (e.g. wind-tunnel airfoil flow with boundary forces), where zero momentum change is not expected
- Only a single snapshot is available with no initial/final pair to difference

## Inputs

- FleCSPH parameter file (.par) copied from the repository's data/ subdirectory with particle count and simulation end time set
- Initial H5part particle snapshot produced by a 3D generator (e.g. noh_3d_generator or sodtube_3d_generator)
- Final H5part particle snapshot written by app/driver/newtonian_3d after FMM-gravity evolution under MPI

## Outputs

- Total linear momentum values (sum of mass*velocity) at initial and final states
- Total angular momentum values (sum of r x m*v) at initial and final states
- momentum_conservation.csv recording initial and final linear and angular momentum values

## How to apply

Build FleCSPH from the cloned repository with cmake flags -DENABLE_UNIT_TESTS=ON and -DLOG_STRIP_LEVEL=1, then 'make -j' and 'make install' to obtain the initial-data generators (e.g. noh_3d_generator, sodtube_3d_generator) and the newtonian_3d driver. Copy a pre-configured parameter file from the repository's data/ subdirectory, set the particle count and simulation end time, and run the generator to produce the initial particle dataset in H5part format. Evolve the system with app/driver/newtonian_3d under MPI using FMM gravity at the implemented first order of the multipole expansion. From both the initial and final H5part snapshots, compute total linear momentum as sum of mass*velocity over all particles and total angular momentum as sum of r x m*v; the rationale is that FMM conserves linear momentum due to the symmetry of nodal interactions, while angular momentum is conserved exactly at the first order of the multipole expansion.

## Related tools

- **FleCSPH** (Provides the FMM gravitational solver, the 3D initial-data generators, and the newtonian_3d driver used to produce the snapshots being checked) — github.com/laristra/flecsph
- **FleCSI** (Supplies the parallel octree data structure over which symmetric nodal interactions give linear-momentum conservation) — github.com/laristra/flecsi
- **H5part** (File format of the initial and final particle snapshots from which momentum sums are computed)
- **MPI** (Backend used to run newtonian_3d for the distributed FMM evolution)

## Examples

```
mpirun -n 4 app/driver/newtonian_3d noh_3d.par
```

## Evaluation signals

- Linear momentum change between initial and final H5part snapshots is ~0, consistent with FMM conserving linear momentum due to symmetric nodal interactions
- Angular momentum change is ~0, as expected because angular momentum is conserved exactly at the first order of the multipole expansion
- Initial and final snapshots contain the same particles (total mass summed from mass*velocity contributions is unchanged)
- momentum_conservation.csv contains both initial and final linear and angular momentum values populated from real snapshot data
- The FMM driver ran at the implemented first order of the multipole expansion, not a higher order, for the angular-momentum claim to hold exactly

## Limitations

- Angular momentum conservation is guaranteed only at the implemented first order of the multipole expansion; higher-order or approximate expansions may break it
- FMM timing (and hence practical run length) strongly depends on the MAC angle, and FMM is approximate compared with exact N-body, which has quadratic complexity
- The current FleCSPH version supports only the MPI backend and does not take advantage of all FleCSI features

## Evidence

- [abstract] FMM conserves linear momentum at first order: "FMM conserves linear momentum. In the implemented first order of the multipole expansion, angular momentum is conserved exactly."
- [readme] FMM gravitational support and octree interactions: "FleCSPH includes support for gravitational forces using the fast multipole method (FMM)."
- [abstract] Complexity and MAC-angle dependence of FMM: "Unlike the exact N-body with quadratic complexity, FMM algorithm has linear (or even sublinear) complexity"
- [readme] Parameter files location for the test problem: "Sample parameter files and the intial data can be found in the `data` subdirectory."
