---
name: mpi-parallel-time-integration
description: Use when a PDE simulation must be time-integrated to a statistically
  saturated state (e.g., t=200) over a multi-dimensional spectral domain (e.g., 256x128x32
  modes) and serial runtime is infeasible, requiring MPI parallelism.
when_to_use_negative:
- The problem fits in a small dense/sparse eigenvalue solve (e.g., Mathieu eigenvalues,
  Rayleigh-Benard growth rates via solve_sparse) where simple dedicated solvers suffice
  without long time integration
- A quick serial integration to early times is adequate; MPI setup overhead (parallel
  HDF5, MPI-FFTW builds) is not justified
- The target system lacks an MPI toolchain (mpiicc/mpiicpc/mpiifort) needed to build
  parallel HDF5 and FFTW support
edam_operation: http://edamontology.org/operation_0244
tools:
- name: Dedalus
  role: Framework providing symbolic equation entry and compiled sparse solvers for
    MPI-parallel time integration
  repo: https://github.com/DedalusProject/dedalus
- name: h5py
  role: Reads the HDF5 snapshot output files produced by the parallel integration
  repo: https://github.com/h5py/h5py
- name: numpy
  role: Array manipulation when post-processing the HDF5 fields
- name: matplotlib
  role: Renders slice plots of potential vorticity and buoyancy perturbation fields
- name: OpenBLAS
  role: Optimized BLAS backend used when building Dedalus from source for the parallel
    environment
  repo: https://github.com/OpenMathLib/OpenBLAS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:568cd10b9e4e2844f54858f542f0f5c5c83f32a07995e62fb2ca3f1c04b9b4b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DedalusProject/dedalus
  tools:
  - Dedalus
  - OpenBLAS
  - Python
  - h5py
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mpi-parallel-time-integration

## Summary

Run long Dedalus spectral simulations (e.g., the 3D quasigeostrophic baroclinic instability example integrated to t=200) as MPI-parallel time integrations using Dedalus's compiled sparse solvers, producing HDF5 snapshot output of fields like potential vorticity (p) and buoyancy (w).

## When to use

Use when a PDE simulation must be time-integrated to a statistically saturated state (e.g., t=200) over a multi-dimensional spectral domain (e.g., 256x128x32 modes) and serial runtime is infeasible, requiring MPI parallelism.

## When NOT to use

- The problem fits in a small dense/sparse eigenvalue solve (e.g., Mathieu eigenvalues, Rayleigh-Benard growth rates via solve_sparse) where simple dedicated solvers suffice without long time integration
- A quick serial integration to early times is adequate; MPI setup overhead (parallel HDF5, MPI-FFTW builds) is not justified
- The target system lacks an MPI toolchain (mpiicc/mpiicpc/mpiifort) needed to build parallel HDF5 and FFTW support

## Inputs

- Dedalus example script (e.g., quasigeostrophic example from dedalus/src/dedalus/examples) with symbolic equation entry
- Spectral domain configuration: 256x128x32 modes for w (buoyancy) and p (potential vorticity)
- Boundary conditions (Ekman pumping) and depth-dependent stratification parameters

## Outputs

- MPI-parallel time-integrated solution reaching t=200 (statistically saturated state)
- HDF5 snapshot files containing the w and p fields at final time
- Matplotlib slice plots showing compact eddy structures

## How to apply

Configure the spectrally-discretized domain (e.g., 256x128x32 modes for the buoyancy w and potential vorticity p variables) and set up the problem with Dedalus's symbolic equation entry, imposing Ekman pumping boundary conditions and depth-dependent stratification in the quasigeostrophic equations. Launch the MPI-parallel time integration with Dedalus's compiled sparse solvers, integrating until t=200 so the flow reaches its statistically saturated state (a sea of compact eddies that form, merge, and break up over several dynamical times). Verify the environment first: HDF5 must be configured with parallel support (`./configure ... CC=mpiicc CXX=mpiicpc F77=mpiifort --enable-shared --enable-parallel`) and FFTW with `--enable-mpi`, and the Dedalus test suite (`python3 -m dedalus test`) must pass. Post-process the resulting HDF5 snapshots with h5py/numpy to extract final-time fields and render slice plots with matplotlib for qualitative validation against the reported example behavior.

## Related tools

- **Dedalus** (Framework providing symbolic equation entry and compiled sparse solvers for MPI-parallel time integration) — https://github.com/DedalusProject/dedalus
- **h5py** (Reads the HDF5 snapshot output files produced by the parallel integration) — https://github.com/h5py/h5py
- **numpy** (Array manipulation when post-processing the HDF5 fields)
- **matplotlib** (Renders slice plots of potential vorticity and buoyancy perturbation fields)
- **OpenBLAS** (Optimized BLAS backend used when building Dedalus from source for the parallel environment) — https://github.com/OpenMathLib/OpenBLAS

## Examples

```
python3 -m dedalus test
```

## Evaluation signals

- The integration runs to t=200 without failure
- HDF5 snapshot files exist at the end of the run and contain both the w and p fields
- Extracted fields show the compact eddies of the statistically saturated state (qualitative expert check against the reported example behavior, Fig. 13)
- The Dedalus test suite passes before launching the production run (`python3 -m dedalus test`)

## Limitations

- MPI parallelism requires separately built dependencies: HDF5 with parallel (--enable-parallel) and FFTW with MPI (--enable-mpi) support, per the article's build instructions
- The saturated-state eddy structure is validated qualitatively (expert visual check), not against a quantitative metric
- Current guidance is that Dedalus runs easily on single AWS instances (e.g., transferred via scp), implying very large-scale runs are not the documented use case

## Evidence

- [abstract] Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods.: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [methods] Build HDF5 with parallel (MPI) support: "./configure --prefix=$BUILD_HOME CC=mpiicc CXX=mpiicpc F77=mpiifort \
                         --enable-shared --enable-parallel"
- [methods] Build FFTW with MPI support: "--enable-shared \
                         --enable-mpi --enable-openmp --enable-threads"
- [methods] Run Dedalus on AWS EC2 instances: "Currently, dedalus can be run easily on single AWS instances."
- [methods] Ensure tests pass: "Ensure tests pass by running:
  sh
  python3 -m dedalus test
  "
