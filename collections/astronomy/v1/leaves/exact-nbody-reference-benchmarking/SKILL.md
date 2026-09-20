---
name: exact-nbody-reference-benchmarking
description: 'Use when benchmarking a fast multipole method (FMM) gravity solver:
  you need a quadratic-complexity exact N-body reference whose wall-clock timings
  prove the FMM runtime scales linearly or sublinearly with particle count N.'
when_to_use_negative:
- The problem is hydrodynamics-only (e.g. Sod shock tube, Noh, Sedov tests) with no
  gravitational forces, so no N-body gravity baseline is relevant
- You need force/accuracy validation rather than timing scaling — the exact N-body
  baseline here is a performance reference, not a correctness check of FMM forces
- FleCSPH has not been built with the MPI backend, which is the only supported runtime
  for the current version
tools:
- name: FleCSPH
  role: Provides the exact N-body gravity computation used as the quadratic-complexity
    reference baseline and the FMM gravity driver under benchmarking
  repo: github.com/laristra/flecsph
- name: FleCSI
  role: Underlying parallel framework supplying the octree data structures on which
    both the exact and FMM gravity paths run
  repo: github.com/laristra/flecsi
- name: Spack
  role: Installs and loads FleCSPH and all dependencies so the benchmark binaries
    (generators and drivers) are available
  repo: github.com/spack/spack
- name: H5part
  role: I/O format for the initial particle distributions fed to the benchmark runs
- name: MPI
  role: Runtime backend under which the benchmark timing runs are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:ff657aa566a146bd9916ca4f2792031da1ea9518191da055f074621c394201ef
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# exact-nbody-reference-benchmarking

## Summary

Runs the exact N-body (quadratic-complexity) gravity computation in FleCSPH as a timing baseline against which the FMM gravity solver's scaling with particle count and sensitivity to the MAC angle can be validated.

## When to use

Use when benchmarking a fast multipole method (FMM) gravity solver: you need a quadratic-complexity exact N-body reference whose wall-clock timings prove the FMM runtime scales linearly or sublinearly with particle count N.

## When NOT to use

- The problem is hydrodynamics-only (e.g. Sod shock tube, Noh, Sedov tests) with no gravitational forces, so no N-body gravity baseline is relevant
- You need force/accuracy validation rather than timing scaling — the exact N-body baseline here is a performance reference, not a correctness check of FMM forces
- FleCSPH has not been built with the MPI backend, which is the only supported runtime for the current version

## Inputs

- FleCSPH initial-data generator outputs for the self-gravity problem at a series of increasing particle counts N (H5part format particle distributions)
- FleCSPH parameter files (e.g. adapted from the pre-configured parameter files in the 'data' subdirectory) specifying gravity method and MAC angle settings
- FleCSPH build with the MPI runtime backend

## Outputs

- Wall-clock timing measurements for the exact N-body (quadratic) gravity runs at each particle count
- Paired FMM timings at each particle count and MAC angle setting
- A combined timing-versus-N and timing-versus-MAC-angle table (e.g. fmm_timing_scaling.csv)

## How to apply

Build FleCSPH either via Spack ('spack install flecsph' then 'spack load flecsph') or by a manual cmake build with the MPI backend (-DFLECSI_RUNTIME_MODEL=mpi; FleCSPH is limited to the MPI backend). Generate initial particle distributions for the self-gravity problem at a series of increasing particle counts N using the FleCSPH initial-data generators, then run the same particle counts with both the exact N-body gravity computation and the newtonian_3d driver using FMM gravity, sweeping several MAC angle settings for the FMM runs. Record wall-clock timings per run and assemble them into a single table of runtime versus particle count and runtime versus MAC angle (e.g. fmm_timing_scaling.csv). The rationale is that the exact N-body method has quadratic complexity, so it serves as the control against which the FMM's linear (or even sublinear) scaling and strong MAC-angle timing sensitivity are demonstrated.

## Related tools

- **FleCSPH** (Provides the exact N-body gravity computation used as the quadratic-complexity reference baseline and the FMM gravity driver under benchmarking) — github.com/laristra/flecsph
- **FleCSI** (Underlying parallel framework supplying the octree data structures on which both the exact and FMM gravity paths run) — github.com/laristra/flecsi
- **Spack** (Installs and loads FleCSPH and all dependencies so the benchmark binaries (generators and drivers) are available) — github.com/spack/spack
- **H5part** (I/O format for the initial particle distributions fed to the benchmark runs)
- **MPI** (Runtime backend under which the benchmark timing runs are executed)

## Examples

```
spack install flecsph && spack load flecsph && mpiexec -n 1 newtonian_3d self_gravity.par
```

## Evaluation signals

- The collected timings must show FMM scaling linearly (or sublinearly) with particle count while the exact N-body reference scales quadratically
- FMM timings must exhibit strong sensitivity to the MAC angle setting
- Timings exist for every particle count N run under both the exact N-body and FMM methods so the complexity comparison is paired
- The timing table includes both runtime versus particle count and runtime versus MAC angle dimensions

## Limitations

- The current version of FleCSPH only supports the MPI backend and does not take advantage of all FleCSI features
- The exact N-body method has quadratic complexity, so feasible particle counts for the reference runs are limited compared to FMM
- Timing results are wall-clock dependent and sensitive to the MAC angle, requiring multiple MAC angle settings to characterize

## Evidence

- [abstract] FMM linear vs exact N-body quadratic scaling: "Unlike the exact N-body with quadratic complexity, FMM algorithm has linear (or even sublinear) complexity"
- [abstract] MAC angle sensitivity of FMM timing: "The timing of FMM algorithm strongly depends on the MAC angle."
- [methods] Spack install/load of FleCSPH: "spack install flecsph"
- [readme] Driver and generator names available after spack load: "You will then have access to the generators and the drivers: 
- sodtube\_{1-2-3}d\_generator, sedov\_{2-3}d\_generator...
- hydro\_{1-2-3}d, newtonian\_3d..."
- [intro] FleCSPH includes FMM gravity support: "FleCSPH includes support
for gravitational forces using the fast multipole method (FMM)."
