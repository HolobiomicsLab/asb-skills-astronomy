---
name: parallel-timing-data-tabulation
description: Use when benchmarking MPI-parallel spectral PDE solvers across core counts
  and you need to tabulate per-run wall-clock timings into efficiency and weak-scaling
  cost tables for validation against reported scaling trends.
when_to_use_negative:
- The target problem is not an IVP solved with global spectral methods over Fourier-Fourier-Chebyshev
  domains — the ~1/log2(N) weak-scaling trend is specific to FFT-based transposes
  in this pencil-decomposition setting
- Only a single core count or single-node run is available, so strong-scaling efficiency
  (which needs a baseline single-node time) cannot be computed
- The benchmark question concerns solver accuracy or eigenvalue correctness rather
  than parallel performance — use validation tests (e.g. python3 -m dedalus test)
  instead of timing tabulation
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: Dedalus
  role: Provides the MPI-parallel 3D Boussinesq hydro and MHD IVP scaling benchmarks
    (Fig. 8) whose wall-clock timings are tabulated
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Numerical computation of parallel efficiencies and weak-scaling costs from
    recorded timings
- name: matplotlib
  role: Plotting strong-scaling efficiency and weak-scaling cost curves against core
    count
- name: h5py
  role: Parallel HDF5 output handling for the benchmark runs, requiring an HDF5 build
    with --enable-parallel MPI support
  repo: https://github.com/h5py/h5py
- name: OpenBLAS
  role: BLAS library built from source as part of the performance-critical Dedalus
    installation used in the benchmarks
  repo: https://github.com/xianyi/OpenBLAS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:27ccb318c6d81428feebe39aaf36ca486d5bf2626c8bbf759304eb0a5bd8f986
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Parallel timing data tabulation

## Summary

A method for tabulating MPI wall-clock timings from parallel scaling benchmarks (strong and weak scaling) of Dedalus's 3D Boussinesq hydrodynamic and MHD IVP problems, computing per-core-count parallel efficiencies and weak-scaling costs, and validating them against reported trends (>50% strong-scaling efficiency down to ~8 pencils per core; weak-scaling cost ~1/log2(N)).

## When to use

Use when benchmarking MPI-parallel spectral PDE solvers across core counts and you need to tabulate per-run wall-clock timings into efficiency and weak-scaling cost tables for validation against reported scaling trends.

## When NOT to use

- The target problem is not an IVP solved with global spectral methods over Fourier-Fourier-Chebyshev domains — the ~1/log2(N) weak-scaling trend is specific to FFT-based transposes in this pencil-decomposition setting
- Only a single core count or single-node run is available, so strong-scaling efficiency (which needs a baseline single-node time) cannot be computed
- The benchmark question concerns solver accuracy or eigenvalue correctness rather than parallel performance — use validation tests (e.g. python3 -m dedalus test) instead of timing tabulation

## Inputs

- Dedalus 3D Boussinesq hydrodynamic IVP scaling benchmark script
- Dedalus 3D MHD IVP scaling benchmark script (Fourier-Fourier-Chebyshev domain)
- MPI-parallel execution environment with parallel HDF5 and FFTW (MPI-enabled) builds
- Set of core counts to sweep (strong-scaling and weak-scaling configurations)

## Outputs

- scaling_benchmark_results.csv with one row per (problem, core count) configuration containing wall-clock timings, strong-scaling parallel efficiencies, and weak-scaling normalized costs
- Per-run wall-clock timing records from each MPI execution
- Strong-scaling efficiency curve per problem (hydro and MHD)
- Weak-scaling cost curve as a function of core count N

## How to apply

Install Dedalus from the deposited repository (pip3 install -r requirements.txt; python3 setup.py build_ext --inplace) on a cluster or cloud instance with parallel HDF5 and FFTW built with MPI support. Launch the hydrodynamic and MHD IVP scaling benchmarks (Fig. 8 of the Dedalus paper) under MPI at each chosen core count, recording wall-clock time per run. Compute strong-scaling parallel efficiency as baseline single-node time divided by n-core time times n, checking the plateau where efficiency stays above ~50% down to roughly 8 pencils per core. Compute weak-scaling normalized cost per core count and verify it follows the expected ~1/log2(N) growth trend for FFT-based computations. Assemble all timings, efficiencies, and weak-scaling costs into one row per (problem, core count) configuration in scaling_benchmark_results.csv.

## Related tools

- **Dedalus** (Provides the MPI-parallel 3D Boussinesq hydro and MHD IVP scaling benchmarks (Fig. 8) whose wall-clock timings are tabulated) — https://github.com/DedalusProject/dedalus
- **numpy** (Numerical computation of parallel efficiencies and weak-scaling costs from recorded timings)
- **matplotlib** (Plotting strong-scaling efficiency and weak-scaling cost curves against core count)
- **h5py** (Parallel HDF5 output handling for the benchmark runs, requiring an HDF5 build with --enable-parallel MPI support) — https://github.com/h5py/h5py
- **OpenBLAS** (BLAS library built from source as part of the performance-critical Dedalus installation used in the benchmarks) — https://github.com/xianyi/OpenBLAS

## Examples

```
mpirun -np N python3 bench/boussinesq_ivp_scaling.py && python3 tabulate_scalings.py --out scaling_benchmark_results.csv
```

## Evaluation signals

- Hydrodynamic strong-scaling parallel efficiency remains above ~50% through the ~8-pencils-per-core regime
- MHD strong-scaling parallel efficiency remains above ~50% in the same regime
- Large 3D problems scale efficiently to thousands of cores
- Weak-scaling normalized cost per core count follows the expected ~1/log2(N) trend for FFT-based computations
- scaling_benchmark_results.csv contains exactly one row per (problem, core count) configuration with non-empty timing, efficiency, and cost fields

## Limitations

- Dedalus can currently be run easily only on single AWS instances; scaling to thousands of cores requires a cluster or suitable cloud configuration with MPI-built HDF5 and FFTW
- Parallel efficiency is expected to degrade as the number of pencils per core falls below roughly 8, since each core no longer has enough work per transpose
- Timings depend on the underlying BLAS/FFTW builds (e.g. MKL vs OpenBLAS); building numpy against MKL or OpenBLAS affects the measured wall-clock times

## Evidence

- [methods] parallel efficiency plateau: [evidence span withheld — anchor status: not_found]
- [methods] build with parallel HDF5: "./configure --prefix=$BUILD_HOME CC=mpiicc CXX=mpiicpc F77=mpiifort \
                         --enable-shared --enable-parallel"
- [methods] FFTW MPI build flags: "--enable-shared \
                         --enable-mpi --enable-openmp --enable-threads"
- [methods] install from source: "pip3 install -r requirements.txt
     python3 setup.py build_ext --inplace"
- [methods] AWS execution scope: "Currently, dedalus can be run easily on single AWS instances."
- [abstract] Dedalus as parallel spectral solver: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [readme] Dedalus description: "Dedalus is a flexible framework for solving partial"
