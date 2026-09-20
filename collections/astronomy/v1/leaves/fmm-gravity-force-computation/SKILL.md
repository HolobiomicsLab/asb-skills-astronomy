---
name: fmm-gravity-force-computation
description: Use when benchmarking long-range self-gravity for large particle systems,
  or when verifying that an FMM implementation scales linearly with particle count
  and quantifying its sensitivity to the MAC angle against a quadratic exact N-body
  baseline.
when_to_use_negative:
- Particle counts are small enough that exact N-body (quadratic) summation is affordable
  and maximum force accuracy is required.
- The analysis targets hydrodynamics problems without self-gravity (e.g., Sod shock
  tubes or Sedov blast waves), where FMM gravity is not exercised.
- The target runtime requires non-MPI backends (Legion or Kokkos node-level parallelism),
  which the current FleCSPH version does not support.
tools:
- name: FleCSPH
  role: SPH application providing the FMM gravity computation, initial-data generators,
    and the newtonian_3d driver to benchmark
  repo: github.com/laristra/flecsph
- name: FleCSI
  role: Compile-time configurable framework supplying the parallel octree data structures
    on which the FMM runs
  repo: github.com/laristra/flecsi
- name: Spack
  role: Package manager used to install and load FleCSPH and its dependencies for
    the benchmark runs
  repo: github.com/spack/spack
- name: MPI
  role: Runtime backend for the distributed tree topology and asynchronous execution
    of the FMM
- name: H5part
  role: I/O format for particle data produced by the benchmark runs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:ccb9e1ebb1f331928679846345068c81cd8d81f5d2c83d12f4cd36fb57ee1f8b
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# fmm-gravity-force-computation

## Summary

Benchmark FleCSPH's fast multipole method (FMM) gravity against an exact N-body reference to demonstrate linear (or sublinear) versus quadratic scaling with particle count, and to characterize the strong dependence of FMM timing on the MAC angle.

## When to use

Use when benchmarking long-range self-gravity for large particle systems, or when verifying that an FMM implementation scales linearly with particle count and quantifying its sensitivity to the MAC angle against a quadratic exact N-body baseline.

## When NOT to use

- Particle counts are small enough that exact N-body (quadratic) summation is affordable and maximum force accuracy is required.
- The analysis targets hydrodynamics problems without self-gravity (e.g., Sod shock tubes or Sedov blast waves), where FMM gravity is not exercised.
- The target runtime requires non-MPI backends (Legion or Kokkos node-level parallelism), which the current FleCSPH version does not support.

## Inputs

- Initial particle distributions for the self-gravity problem at increasing particle counts N, generated with the FleCSPH initial-data generators
- Pre-configured parameter files from the flecsph 'data' subdirectory (e.g., edited .par input files)
- MAC angle settings to sweep

## Outputs

- fmm_timing_scaling.csv: runtime versus particle count and runtime versus MAC angle for both FMM and exact N-body runs
- H5part-format simulation output files per run

## How to apply

Build FleCSPH via Spack ('spack install flecsph' then 'spack load flecsph') or a manual cmake build with the MPI backend (-DFLECSI_RUNTIME_MODEL=mpi), since the current version only supports the MPI runtime. Use the FleCSPH initial-data generators to produce particle distributions for the self-gravity problem at a series of increasing particle counts N, then run the newtonian_3d evolution driver with FMM gravity for each N and for several MAC angle settings, recording wall-clock timings per run (I/O in H5part format). Run the exact N-body (quadratic-complexity) gravity computation for the same particle counts as the reference baseline. Assemble runtime versus particle count and runtime versus MAC angle into fmm_timing_scaling.csv. The decision point is the validation: FMM must scale linearly (or sublinearly) while the exact reference scales quadratically, and FMM timings must show strong sensitivity to the MAC angle.

## Related tools

- **FleCSPH** (SPH application providing the FMM gravity computation, initial-data generators, and the newtonian_3d driver to benchmark) — github.com/laristra/flecsph
- **FleCSI** (Compile-time configurable framework supplying the parallel octree data structures on which the FMM runs) — github.com/laristra/flecsi
- **Spack** (Package manager used to install and load FleCSPH and its dependencies for the benchmark runs) — github.com/spack/spack
- **MPI** (Runtime backend for the distributed tree topology and asynchronous execution of the FMM)
- **H5part** (I/O format for particle data produced by the benchmark runs)

## Examples

```
spack install flecsph; spack load flecsph
```

## Evaluation signals

- Collected timings show FMM scaling linearly (or sublinearly) with particle count while the exact N-body reference scales quadratically.
- FMM timings exhibit strong sensitivity to the MAC angle across the swept settings.
- FMM conserves linear momentum, and angular momentum is conserved exactly at the implemented first order of the multipole expansion.
- fmm_timing_scaling.csv contains runtime rows for every (particle count, MAC angle) combination for both FMM and exact reference runs.
- All runs complete successfully under the MPI-only backend with H5part output written for each configuration.

## Limitations

- The current version of FleCSPH is limited to the MPI backend and does not take advantage of all FleCSI features (e.g., Legion/Kokkos runtimes).
- The timing of the FMM algorithm strongly depends on the MAC angle, so timings must be reported per MAC angle setting and cannot be summarized by a single scaling curve.
- FMM is approximate: only at the implemented first order of the multipole expansion is angular momentum conserved exactly, so it is not interchangeable with the exact N-body reference for accuracy-critical studies.

## Evidence

- [abstract] Unlike the exact N-body with quadratic complexity, FMM algorithm has linear (or even sublinear) complexity: "Unlike the exact N-body with quadratic complexity, FMM algorithm has linear (or even sublinear) complexity"
- [abstract] The timing of FMM algorithm strongly depends on the MAC angle.: "The timing of FMM algorithm strongly depends on the MAC angle."
- [intro] FleCSPH includes support for gravitational forces using the fast multipole method (FMM).: "FleCSPH includes support
for gravitational forces using the fast multipole method (FMM)."
- [abstract] FMM conserves linear momentum; angular momentum conserved exactly at first order.: "FMM conserves linear momentum. In the implemented first order of the multipole expansion, angular momentum is conserved exactly."
- [methods] Spack install/load of FleCSPH to access generators and drivers.: "spack install flecsph"
- [readme] README lists Newtonian-gravity examples available in FleCSPH.: "single and binary stars with Newtonian gravity in 3D."
