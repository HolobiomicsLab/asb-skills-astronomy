---
name: mpi-parallel-job-execution
description: Use when a FleCSPH executable (hydro_1-2-3d, newtonian_3d, or a *_generator)
  must be run on multiple MPI ranks to produce H5part particle data or timing measurements
  for tree-based particle simulations.
when_to_use_negative:
- If the target application requires a task-based runtime other than MPI (e.g. Legion
  or Kokkos backends) — the current version of FleCSPH is limited to only the MPI
  backend
- For post-processing or analysis of already-written H5part snapshots (e.g. computing
  momentum conservation sums), which needs no parallel job execution
- 'For non-FleCSPH codes: the Spack install/load workflow here only exposes FleCSPH''s
  generators (sodtube_1-2-3d_generator, sedov_2-3d_generator, ...) and drivers (hydro_1-2-3d,
  newtonian_3d)'
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: FleCSPH
  role: The MPI application whose generators and drivers (newtonian_3d, hydro_1-2-3d,
    *_generator) are launched as parallel jobs
  repo: github.com/laristra/flecsph
- name: Spack
  role: Installs FleCSPH with all dependencies and loads the executables into the
    environment via spack install flecsph / spack load flecsph
  repo: github.com/spack/spack
- name: FleCSI
  role: Compile-time configurable parallel framework providing the MPI backend and
    parallel octree data structures that distribute the particle work across ranks
  repo: github.com/laristra/flecsi
- name: MPI
  role: Execution runtime under which FleCSPH drivers are launched (the only supported
    backend)
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:da98054c866b5aac2b36bbed1ab5ee66718348fa18441e74ec4eb7a8366a6cc0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FleCSI
  - FleCSPH
  - MPI
  - Spack
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mpi-parallel-job-execution

## Summary

Executes FleCSPH's generators and drivers (e.g. newtonian_3d, noh_3d_generator) as MPI parallel jobs after installing the package with Spack; FleCSPH currently only supports the MPI backend, so all runs go through MPI ranks rather than task-based runtimes.

## When to use

Use when a FleCSPH executable (hydro_1-2-3d, newtonian_3d, or a *_generator) must be run on multiple MPI ranks to produce H5part particle data or timing measurements for tree-based particle simulations.

## When NOT to use

- If the target application requires a task-based runtime other than MPI (e.g. Legion or Kokkos backends) — the current version of FleCSPH is limited to only the MPI backend
- For post-processing or analysis of already-written H5part snapshots (e.g. computing momentum conservation sums), which needs no parallel job execution
- For non-FleCSPH codes: the Spack install/load workflow here only exposes FleCSPH's generators (sodtube_1-2-3d_generator, sedov_2-3d_generator, ...) and drivers (hydro_1-2-3d, newtonian_3d)

## Inputs

- FleCSPH parameter file (.par) copied from the repository's data/ subdirectory, edited for particle count and simulation end time
- FleCSPH installation accessible via spack load flecsph (or a manual cmake/Make install with -DFLECSI_RUNTIME_MODEL=mpi)
- An MPI launcher (mpirun/srun) with the requested number of ranks
- Optional: pre-generated H5part initial particle dataset from a *_generator run

## Outputs

- Evolved particle snapshots in H5part format (initial and final states)
- Generator-produced initial particle datasets in H5part format
- Per-run wall-clock timings suitable for assembling runtime-versus-particle-count and runtime-versus-MAC-angle tables (e.g. fmm_timing_scaling.csv)

## How to apply

First make the executables available: install the package with 'spack install flecsph' (or the manual cmake route with -DFLECSI_RUNTIME_MODEL=mpi, unit tests on, then 'make -j' and 'make install'), then 'spack load flecsph' to put the generators and drivers on PATH. Choose the driver matching the physics problem — newtonian_3d for 3D self-gravity with FMM, or hydro_{1-2-3}d for pure SPH — and supply a pre-configured parameter file copied from the repository's data/ subdirectory, edited for the desired particle count and end time. Launch the driver under MPI (mpirun/mpirun -n N) so the FleCSI MPI backend distributes the parallel octree across ranks; run initial-data generators the same way to write H5part input datasets. For scaling studies, repeat launches at increasing particle counts and multiple MAC angle settings, recording wall-clock timings per run, and keep the exact N-body reference runs on the same launch mechanism for a fair baseline. Rationale: the asynchronous distributed tree topology plus FMM is what the MPI runtime must schedule, so correct rank counts and a valid parameter file are the key decision points.

## Related tools

- **FleCSPH** (The MPI application whose generators and drivers (newtonian_3d, hydro_1-2-3d, *_generator) are launched as parallel jobs) — github.com/laristra/flecsph
- **Spack** (Installs FleCSPH with all dependencies and loads the executables into the environment via spack install flecsph / spack load flecsph) — github.com/spack/spack
- **FleCSI** (Compile-time configurable parallel framework providing the MPI backend and parallel octree data structures that distribute the particle work across ranks) — github.com/laristra/flecsi
- **MPI** (Execution runtime under which FleCSPH drivers are launched (the only supported backend))

## Examples

```
spack load flecsph; mpirun -n 8 app/driver/newtonian_3d noh_3d.par
```

## Evaluation signals

- 'spack load flecsph' puts the generators (sodtube_1-2-3d_generator, sedov_2-3d_generator, ...) and drivers (hydro_1-2-3d, newtonian_3d) on the executable path
- The driver completes across all requested MPI ranks without rank failures and writes H5part snapshots for both initial and final states
- The parameter file reflects the intended particle count and end time set in step 2 of the workflow before launch
- Recorded wall-clock timings across repeated runs at increasing N show FMM scaling linearly/sublinearly while the exact N-body baseline scales quadratically, and FMM timing is sensitive to the MAC angle
- The build used the MPI runtime model (e.g. -DFLECSI_RUNTIME_MODEL=mpi), not a task-based backend

## Limitations

- The current version of FleCSPH only supports the MPI backend and does not take advantage of all FleCSI features (no Legion/Kokkos runtimes)
- Timing results are launch-mechanism dependent: FMM timing strongly depends on the MAC angle, so comparisons across runs must fix or record this parameter
- Cinch-based builds prohibit in-place builds; an out-of-source build directory is required for the manual cmake route

## Evidence

- [abstract] MPI backend limitation: "the current version of FleCSPH does not take advantage of all features provided in FleCSI: it is limited to only having the support for the MPI backend."
- [readme] Spack install and load flow: "This will build all the dependencies, compile and install FleCSPH."
- [methods] Manual clone for driver access: "git clone --recursive git@github.com:laristra/flecsph.git"
- [readme] Parameter file workflow: "Sample parameter files and the intial data can be found in the `data` subdirectory."
