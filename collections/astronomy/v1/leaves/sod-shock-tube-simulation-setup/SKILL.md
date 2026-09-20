---
name: sod-shock-tube-simulation-setup
description: Use when a hydrodynamic validation or benchmark run is needed — e.g.
  a shock-capturing test case with identical initial conditions across two builds
  (such as comparing the MPI backend against a Legion or Kokkos backend) using FleCSPH's
  Sod tube generators and drivers.
when_to_use_negative:
- The problem is not a shock-capturing test — FleCSPH offers other cases (Noh shock
  test, Sedov blast waves, airfoil wind tunnel, Newtonian-gravity binaries) that need
  their own generators and drivers.
- The goal is a quick install-and-run without code or backend changes — the Spack
  route (spack install flecsph) avoids the manual double-build workflow required for
  backend comparison.
- A dimensionality that the Sod tube driver does not support is required (e.g. 1D
  airflow or pressure-driven spherical implosion, which are 2D/3D only).
edam_operation: http://edamontology.org/operation_3096
tools:
- name: FleCSPH
  role: SPH simulation tool providing the Sod tube generators and hydro drivers used
    to set up and evolve the shock tube problem
  repo: github.com/laristra/flecsph
- name: FleCSI
  role: Compile-time configurable framework supplying the parallel octree data structures
    that FleCSPH exercises; built from the stable/flecsph branch first
  repo: github.com/laristra/flecsi
- name: H5part
  role: I/O format for the generated initial data and evolved particle trajectories
- name: Spack
  role: Alternative installation path that builds all dependencies and installs FleCSPH
    (spack install flecsph; spack load flecsph)
  repo: github.com/spack/spack
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:3d0264d468217475db2cdb6fbfb64fe6b7f02bb442823a110d1d8c1eaf7bad3a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FleCSI
  - FleCSPH
  - H5part
  - Kokkos
  - Legion
  - MPI
  - Spack
  edam_operation: http://edamontology.org/operation_3096
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# sod-shock-tube-simulation-setup

## Summary

Configure, generate initial data for, and run a 1D/2D/3D Sod shock tube simulation with FleCSPH's SPH solver on the FleCSI octree infrastructure, using the pre-configured parameter files (e.g. sodtube_t1_n1000.par) from the data directory and the sodtube/hydro executables.

## When to use

Use when a hydrodynamic validation or benchmark run is needed — e.g. a shock-capturing test case with identical initial conditions across two builds (such as comparing the MPI backend against a Legion or Kokkos backend) using FleCSPH's Sod tube generators and drivers.

## When NOT to use

- The problem is not a shock-capturing test — FleCSPH offers other cases (Noh shock test, Sedov blast waves, airfoil wind tunnel, Newtonian-gravity binaries) that need their own generators and drivers.
- The goal is a quick install-and-run without code or backend changes — the Spack route (spack install flecsph) avoids the manual double-build workflow required for backend comparison.
- A dimensionality that the Sod tube driver does not support is required (e.g. 1D airflow or pressure-driven spherical implosion, which are 2D/3D only).

## Inputs

- Pre-configured Sod tube parameter file ../data/sodtube_t1_n1000.par (n1000, t1 configuration)
- FleCSPH installation built against FleCSI (stable/flecsph branch) with the desired runtime backend (MPI, or task-based backend)
- sodtube_1d_generator executable from app/id_generators

## Outputs

- sodtube.par working parameter file
- Initial H5part particle dataset for the 1D Sod tube
- Evolved hydrodynamics trajectory (H5part) after running hydro_1d to the parameter-file end time
- Wall-clock execution time / timing table (e.g. backend_timing_comparison.csv) when used for backend benchmarks

## How to apply

First obtain FleCSPH, either via Spack (spack install flecsph; spack load flecsph) or by building from source: clone FleCSI at the stable/flecsph branch with submodules, configure it with cmake (-DENABLE_OPENMP=OFF, -DENABLE_METIS=ON, C++17), build with make -j, then clone FleCSPH recursively and configure with -DENABLE_UNIT_TESTS=ON before make -j and make install. Copy the pre-configured parameter file ../data/sodtube_t1_n1000.par to sodtube.par and edit only the parameters under study (keeping particle count and physics parameters fixed for like-for-like runs). Generate the initial particle data with the sodtube_1d_generator from app/id_generators, then run the evolution driver app/driver/hydro_1d with sodtube.par to evolve to the end time in the parameter file. Output is written in the H5part format for use with modern visualization software, so both backend runs should produce comparable trajectory files plus wall-clock timings for comparison.

## Related tools

- **FleCSPH** (SPH simulation tool providing the Sod tube generators and hydro drivers used to set up and evolve the shock tube problem) — github.com/laristra/flecsph
- **FleCSI** (Compile-time configurable framework supplying the parallel octree data structures that FleCSPH exercises; built from the stable/flecsph branch first) — github.com/laristra/flecsi
- **H5part** (I/O format for the generated initial data and evolved particle trajectories)
- **Spack** (Alternative installation path that builds all dependencies and installs FleCSPH (spack install flecsph; spack load flecsph)) — github.com/spack/spack

## Examples

```
cp ../data/sodtube_t1_n1000.par sodtube.par; cmake .. -DCMAKE_BUILD_TYPE=debug -DENABLE_UNIT_TESTS=ON; make -j; make install
```

## Evaluation signals

- Both runs (reference and comparison, e.g. MPI vs Legion/Kokkos backend) complete the hydro evolution to the end time specified in the parameter file.
- Particle count and all physics parameters are identical between the two runs for a like-for-like comparison (only the runtime backend differs).
- The comparison table (e.g. backend_timing_comparison.csv) contains timing entries for both backends for the same Sod tube configuration.
- CMake configure reports ENABLE_UNIT_TESTS=ON for the FleCSPH build so unit tests can be run before production runs.
- Output files are valid H5part and readable by modern visualization software.

## Limitations

- The current version of FleCSPH only supports the MPI backend and does not take advantage of all FleCSI features; task-based backends (Legion distributed, Kokkos node-level) must be enabled by rebuilding FleCSI and FleCSPH with the alternative runtime backend.
- FleCSPH must be built against a FleCSI checkout from the stable/flecsph branch with submodules updated; arbitrary FleCSI versions are not covered by the documented workflow.
- Timing comparisons for FMM/tree operations are sensitive to runtime parameters such as the MAC angle, so unrelated parameter drift can confound backend benchmarks.

## Evidence

- [methods] cp ../data/sodtube_t1_n1000.par sodtube.par: [evidence span withheld — anchor status: not_found]
- [methods] cmake .. -DENABLE_OPENMP=OFF -DCXX_CONFORMANCE_STANDARD=c++17 -DENABLE_METIS=ON: "cmake .. \
 -DENABLE_OPENMP=OFF \
 -DCXX_CONFORMANCE_STANDARD=c++17 \
 -DENABLE_METIS=ON"
- [methods] cmake .. -DCMAKE_BUILD_TYPE=debug -DENABLE_UNIT_TESTS=ON: "cmake .. \
 -DCMAKE_BUILD_TYPE=debug \
 -DENABLE_UNIT_TESTS=ON"
- [abstract] limited to only having the support for the MPI backend: "it is limited to only having the support for the MPI backend"
- [readme] Sod shock tubes in 1D/2D/3D and H5part I/O: "We provide several examples of physics problems in 1D, 2D and 3D:

- Sod shock tubes in 1D/2D/3D;"
- [readme] Sample parameter files and initial data location: "Sample parameter files and the intial data can be found in the `data` subdirectory."
