---
name: runtime-scaling-analysis
description: Use when you need to empirically verify the asymptotic scaling of a numerical
  kernel (e.g., FMM vs. direct summation) by sweeping particle count N and a tuning
  parameter such as the multipole acceptance criterion (MAC) angle.
when_to_use_negative:
- When the goal is verifying physical correctness of gravity results (momentum or
  energy conservation) rather than performance scaling
- When particle count is fixed and only a single MAC angle is of interest — a single
  timing cannot distinguish linear from quadratic complexity
- When using runtime backends other than MPI, since the current FleCSPH version only
  supports the MPI backend
edam_operation: http://edamontology.org/operation_3875
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: Spack
  role: Package manager used to install and load FleCSPH and its dependencies before
    running the timing sweep
  repo: github.com/spack/spack
- name: FleCSPH
  role: SPH application providing the newtonian_3d driver, FMM gravity, initial-data
    generators, and the exact N-body reference
  repo: github.com/laristra/flecsph
- name: FleCSI
  role: Compile-time configurable framework supplying the parallel octree data structures
    underlying the FMM computation being timed
  repo: github.com/laristra/flecsi
- name: MPI
  role: The only supported FleCSI runtime backend for the current FleCSPH version
    used to execute distributed timing runs
- name: H5part
  role: I/O format for particle data consumed by generators and drivers during the
    timing experiments
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:fa87e39f400ebe918cf30c49f34bf75ebe7db93de41a83b4b43dc9453f579fa8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FleCSI
  - FleCSPH
  - H5part
  - MPI
  - Spack
  edam_operation: http://edamontology.org/operation_3875
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# runtime-scaling-analysis

## Summary

Empirically characterizes algorithmic complexity by timing FleCSPH's FMM gravity computation against an exact N-body (quadratic) reference across increasing particle counts and multiple MAC angle settings, producing a runtime-scaling table.

## When to use

Use when you need to empirically verify the asymptotic scaling of a numerical kernel (e.g., FMM vs. direct summation) by sweeping particle count N and a tuning parameter such as the multipole acceptance criterion (MAC) angle.

## When NOT to use

- When the goal is verifying physical correctness of gravity results (momentum or energy conservation) rather than performance scaling
- When particle count is fixed and only a single MAC angle is of interest — a single timing cannot distinguish linear from quadratic complexity
- When using runtime backends other than MPI, since the current FleCSPH version only supports the MPI backend

## Inputs

- Particle initial distributions for the self-gravity problem at increasing particle counts N, generated with FleCSPH initial-data generators (H5part format)
- FleCSPH parameter files (.par) configured for FMM gravity with several MAC angle settings
- Exact N-body (direct summation) gravity runs at the same particle counts as the reference baseline

## Outputs

- fmm_timing_scaling.csv: a table of wall-clock runtime versus particle count and runtime versus MAC angle for both FMM and exact N-body
- Scaling characterization: FMM linear (or sublinear) in N vs. exact N-body quadratic in N, plus MAC-angle sensitivity of FMM timings

## How to apply

Build and install FleCSPH with the MPI runtime backend (either `spack install flecsph` + `spack load flecsph`, or a manual cmake build with -DFLECSI_RUNTIME_MODEL=mpi). Use the FleCSPH initial-data generators to produce self-gravity particle distributions at a series of increasing particle counts N. For each N and for several MAC angle settings, run the `newtonian_3d` driver with FMM gravity and record wall-clock timings; run the exact N-body (quadratic-complexity) gravity computation at the same particle counts as the baseline. Assemble all timings into a single table of runtime versus particle count and runtime versus MAC angle, written to fmm_timing_scaling.csv. The rationale: FMM should scale linearly or sublinearly while the exact N-body reference scales quadratically, and FMM timing should be strongly sensitive to the MAC angle because the opening angle controls how many cells pass the multipole acceptance criterion.

## Related tools

- **Spack** (Package manager used to install and load FleCSPH and its dependencies before running the timing sweep) — github.com/spack/spack
- **FleCSPH** (SPH application providing the newtonian_3d driver, FMM gravity, initial-data generators, and the exact N-body reference) — github.com/laristra/flecsph
- **FleCSI** (Compile-time configurable framework supplying the parallel octree data structures underlying the FMM computation being timed) — github.com/laristra/flecsi
- **MPI** (The only supported FleCSI runtime backend for the current FleCSPH version used to execute distributed timing runs)
- **H5part** (I/O format for particle data consumed by generators and drivers during the timing experiments)

## Examples

```
spack install flecsph && spack load flecsph; ./newtonian_3d newtonian.par  # repeat over increasing N and several MAC angles; exact N-body runs as baseline; write fmm_timing_scaling.csv
```

## Evaluation signals

- Fitted runtime-versus-N curve for FMM scales linearly or sublinearly, while the exact N-body reference curve scales quadratically
- FMM timings show strong sensitivity to the MAC angle across the swept settings
- Timings exist for every (N, MAC angle) combination and for the exact N-body baseline at the same particle counts
- All timings are assembled into fmm_timing_scaling.csv with columns for particle count and MAC angle

## Limitations

- The current version of FleCSPH only supports the MPI backend and does not take advantage of all FleCSI features, so timings reflect that configuration only
- The implemented FMM uses only the first order of the multipole expansion, which bounds the accuracy/performance trade-off explored via the MAC angle
- FMM timing depends strongly on the MAC angle, so a single angle setting gives an incomplete or misleading scaling picture

## Evidence

- [abstract] FMM has linear (or even sublinear) complexity unlike exact N-body with quadratic complexity: "Unlike the exact N-body with quadratic complexity, FMM algorithm has linear (or even sublinear) complexity"
- [abstract] FMM timing strongly depends on the MAC angle: "The timing of FMM algorithm strongly depends on the MAC angle."
- [intro] FleCSPH includes FMM gravity support executed on FleCSI's parallel octree: "FleCSPH includes support
for gravitational forces using the fast multipole method (FMM)."
- [readme] Spack install and load commands expose the generators and drivers: "spack install flecsph 
```
This will build all the dependencies, compile and install FleCSPH. 
In order to use FleCSPH executables simply run: 
```{engine=sh}
spack load flecsph"
- [readme] newtonian_3d driver and generators are available after spack load: "- sodtube\_{1-2-3}d\_generator, sedov\_{2-3}d\_generator...
- hydro\_{1-2-3}d, newtonian\_3d..."
- [abstract] Current FleCSPH limited to MPI backend: "it is limited to only having the support for the MPI backend"
