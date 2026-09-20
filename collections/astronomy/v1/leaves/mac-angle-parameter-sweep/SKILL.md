---
name: mac-angle-parameter-sweep
description: Use when benchmarking a fast multipole method's runtime scaling against
  an exact N-body reference, or when quantifying how the MAC opening angle controls
  the FMM accuracy/performance trade-off in tree-based particle codes.
when_to_use_negative:
- The goal is hydrodynamic accuracy testing (e.g. Sod shock tube or Sedov blast validation)
  rather than gravitational solver performance benchmarking.
- The particle count is fixed and only solution accuracy (e.g. momentum or angular
  momentum conservation) is of interest, so a timing sweep adds no information.
- The application requires runtime backends other than MPI, since the current FleCSPH
  version is limited to the MPI backend.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: FleCSPH
  role: SPH application providing the newtonian_3d driver with FMM gravity and the
    exact N-body reference used for the timing sweep
  repo: github.com/laristra/flecsph
- name: Spack
  role: Package manager used to install and load FleCSPH and its dependencies
  repo: github.com/spack/spack
- name: FleCSI
  role: Underlying parallel octree data-structure framework on which the FMM gravity
    computation runs
  repo: github.com/laristra/flecsi
- name: MPI
  role: Distributed runtime backend required by the current FleCSPH version for parallel
    execution of the benchmark runs
- name: H5part
  role: I/O format used by FleCSPH for particle data input/output
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:e57e5b6ca79d6d314b88416e0dba5765d24ee378562cd7091809b35880718610
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FleCSI
  - FleCSPH
  - H5part
  - MPI
  - Spack
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mac-angle-parameter-sweep

## Summary

Sweeps the multipole-acceptance criterion (MAC) angle in FleCSPH's FMM gravity solver across a series of particle counts, recording wall-clock timings to demonstrate FMM's linear/sublinear scaling versus the quadratic exact N-body baseline and its strong timing sensitivity to the MAC angle.

## When to use

Use when benchmarking a fast multipole method's runtime scaling against an exact N-body reference, or when quantifying how the MAC opening angle controls the FMM accuracy/performance trade-off in tree-based particle codes.

## When NOT to use

- The goal is hydrodynamic accuracy testing (e.g. Sod shock tube or Sedov blast validation) rather than gravitational solver performance benchmarking.
- The particle count is fixed and only solution accuracy (e.g. momentum or angular momentum conservation) is of interest, so a timing sweep adds no information.
- The application requires runtime backends other than MPI, since the current FleCSPH version is limited to the MPI backend.

## Inputs

- FleCSPH initial particle distributions for the self-gravity problem at increasing particle counts N (generated with the FleCSPH initial-data generators, e.g. *_generator executables)
- Input parameter files (.par) for the newtonian_3d driver, varied over MAC angle settings
- Exact N-body reference runs at the same particle counts

## Outputs

- fmm_timing_scaling.csv containing runtime versus particle count and runtime versus MAC angle
- Timing tables demonstrating FMM linear/sublinear scaling versus quadratic exact N-body scaling

## How to apply

Install FleCSPH via Spack (`spack install flecsph`, `spack load flecsph`) or a manual CMake build configured with the MPI backend (`-DFLECSI_RUNTIME_MODEL=mpi`). Use the FleCSPH initial-data generators to produce self-gravity particle distributions at a series of increasing particle counts N. For each N and for several MAC angle settings, run the `newtonian_3d` driver with FMM gravity and record wall-clock timings; then run the exact N-body (quadratic-complexity) gravity computation on the same particle counts as the reference baseline. Assemble all timings into a single table of runtime versus particle count and runtime versus MAC angle, written to fmm_timing_scaling.csv. The rationale is that FMM, unlike exact N-body, has linear (or even sublinear) complexity, and its timing depends strongly on the MAC angle, so both dependencies must be captured in one sweep.

## Related tools

- **FleCSPH** (SPH application providing the newtonian_3d driver with FMM gravity and the exact N-body reference used for the timing sweep) — github.com/laristra/flecsph
- **Spack** (Package manager used to install and load FleCSPH and its dependencies) — github.com/spack/spack
- **FleCSI** (Underlying parallel octree data-structure framework on which the FMM gravity computation runs) — github.com/laristra/flecsi
- **MPI** (Distributed runtime backend required by the current FleCSPH version for parallel execution of the benchmark runs)
- **H5part** (I/O format used by FleCSPH for particle data input/output)

## Examples

```
spack install flecsph && spack load flecsph && mpirun -np 4 newtonian_3d newtonian.par
```

## Evaluation signals

- The collected timings show FMM scaling linearly (or sublinearly) with particle count.
- The exact N-body reference timings scale quadratically with particle count.
- FMM timings exhibit strong sensitivity to the MAC angle across the swept settings.
- All runs for a given particle count use the same initial particle distribution and differ only in gravity method or MAC angle, so the timing comparison is apples-to-apples.
- fmm_timing_scaling.csv contains runtime entries for every (particle count, MAC angle, gravity method) combination executed.

## Limitations

- The current version of FleCSPH only supports the MPI backend and does not take advantage of all FleCSI features.
- The FMM is implemented only to the first order of the multipole expansion; this limits achievable accuracy at large MAC angles even though angular momentum is conserved exactly at that order.
- Timing results depend on machine and MPI configuration; wall-clock comparisons are only meaningful within a consistent environment.

## Evidence

- [abstract] FMM has linear (or even sublinear) complexity unlike exact N-body with quadratic complexity.: "Unlike the exact N-body with quadratic complexity, FMM algorithm has linear (or even sublinear) complexity"
- [abstract] FMM timing strongly depends on the MAC angle.: "The timing of FMM algorithm strongly depends on the MAC angle."
- [methods] Install FleCSPH and dependencies with Spack.: "spack install flecsph"
- [intro] FleCSPH includes FMM gravity support on the FleCSI octree.: "FleCSPH includes support for gravitational forces using the fast multipole method (FMM)."
- [readme] The README lists newtonian_3d among the available drivers.: "- sodtube\_{1-2-3}d\_generator, sedov\_{2-3}d\_generator... - hydro\_{1-2-3}d, newtonian\_3d..."
