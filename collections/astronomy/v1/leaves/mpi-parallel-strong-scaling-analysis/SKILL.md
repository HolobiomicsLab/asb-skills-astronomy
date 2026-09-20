---
name: mpi-parallel-strong-scaling-analysis
description: Use when you must quantify how an MPI-parallel PDE solver scales to large
  core counts — i.e., you have per-run timings for fixed-size (strong) or fixed-per-rank
  (weak) 3D spectral problems and need parallel-efficiency curves.
when_to_use_negative:
- The workload is a serial or OpenMP-only code with no MPI domain decomposition (pencil/transform
  layout), so no per-core-count MPI timings exist
- The goal is correctness verification rather than performance — the Dedalus test
  suite (`python3 -m dedalus test`) and eigenvalue comparisons (e.g., waves-on-a-string
  relative error) are the right tool
- The problem is small enough to fit comfortably on one node, so parallel efficiency
  below the ~8-pencils-per-core regime is uninformative
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Dedalus
  role: MPI-parallel spectral PDE solver whose hydro/MHD IVP scaling benchmarks are
    executed and timed
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: numerical arrays and timing/efficiency arithmetic used to build the scaling
    tables
- name: matplotlib
  role: plotting strong-scaling efficiency and weak-scaling cost curves
- name: h5py
  role: reading benchmark output written as HDF5 (parallel HDF5 built with --enable-parallel)
  repo: https://github.com/h5py/h5py
- name: OpenBLAS
  role: BLAS backend linked into the Dedalus build used for the benchmarks
  repo: https://github.com/OpenMathLib/OpenBLAS
- name: scipy
  role: Python scientific stack dependency of the Dedalus build
  repo: https://github.com/scipy/scipy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:a483b60bb85e7c926c16853dc02490f2285c8fe70bd36684dc33758e81437880
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
  - scipy
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# mpi-parallel-strong-scaling-analysis

## Summary

Benchmarks MPI-parallel scientific codes (e.g., Dedalus's 3D Boussinesq hydro/MHD IVP benchmarks) by measuring wall-clock times across core counts, computing strong-scaling parallel efficiency and weak-scaling cost normalized per core, and validating the expected 1/log2(N) weak-scaling trend of FFT-based spectral methods.

## When to use

Use when you must quantify how an MPI-parallel PDE solver scales to large core counts — i.e., you have per-run timings for fixed-size (strong) or fixed-per-rank (weak) 3D spectral problems and need parallel-efficiency curves.

## When NOT to use

- The workload is a serial or OpenMP-only code with no MPI domain decomposition (pencil/transform layout), so no per-core-count MPI timings exist
- The goal is correctness verification rather than performance — the Dedalus test suite (`python3 -m dedalus test`) and eigenvalue comparisons (e.g., waves-on-a-string relative error) are the right tool
- The problem is small enough to fit comfortably on one node, so parallel efficiency below the ~8-pencils-per-core regime is uninformative

## Inputs

- Dedalus IVP benchmark scripts for 3D Boussinesq hydrodynamic and MHD problems (Fourier-Fourier-Chebyshev domains)
- Cluster, cloud (AWS EC2), or single-node MPI environment with parallel HDF5 and FFTW built with MPI support
- Per-run wall-clock timing measurements at each core count
- Baseline single-node (or single-node-group) timing for the strong-scaling normalization

## Outputs

- scaling_benchmark_results.csv with one row per (problem, core count) configuration
- Strong-scaling parallel-efficiency curves (efficiency = T_baseline / (n * T_n)) vs. core count
- Weak-scaling normalized cost per core, checked against the ~1/log2(N) trend
- Plots (e.g., via matplotlib) of efficiency and cost vs. core count

## How to apply

Install the code from the deposited repository per the documented build (pip3 install -r requirements.txt; python3 setup.py build_ext --inplace) with parallel HDF5 (--enable-shared --enable-parallel) and MPI-enabled FFTW (--enable-mpi --enable-openmp --enable-threads), then run `python3 -m dedalus test` to validate the build. Launch MPI executions of the hydrodynamic and MHD IVP scaling benchmarks (e.g., the Fig. 8 3D Boussinesq problems in Fourier-Fourier-Chebyshev domains) across a ladder of core counts on a cluster or AWS EC2 instances. For each run record wall-clock time and compute strong-scaling efficiency as baseline single-node time divided by n-core time, multiplied by n; verify efficiency stays above ~50% down to roughly 8 pencils per core. For weak scaling, compute the normalized cost per core count and check it grows approximately as 1/log2(N), the expected behavior for FFT-based global spectral methods. Assemble timings, efficiencies, and weak-scaling costs into a CSV with one row per (problem, core count) configuration.

## Related tools

- **Dedalus** (MPI-parallel spectral PDE solver whose hydro/MHD IVP scaling benchmarks are executed and timed) — https://github.com/DedalusProject/dedalus
- **numpy** (numerical arrays and timing/efficiency arithmetic used to build the scaling tables)
- **matplotlib** (plotting strong-scaling efficiency and weak-scaling cost curves)
- **h5py** (reading benchmark output written as HDF5 (parallel HDF5 built with --enable-parallel)) — https://github.com/h5py/h5py
- **OpenBLAS** (BLAS backend linked into the Dedalus build used for the benchmarks) — https://github.com/OpenMathLib/OpenBLAS
- **scipy** (Python scientific stack dependency of the Dedalus build) — https://github.com/scipy/scipy

## Examples

```
pip3 install -r requirements.txt; python3 setup.py build_ext --inplace; python3 -m dedalus test; mpirun -n 128 python3 -m dedalus/examples/ivp/boussinesq_mhd
```

## Evaluation signals

- Strong-scaling parallel efficiency (T_baseline / (n * T_n)) remains above ~50% down to roughly 8 pencils per core for both hydro and MHD problems
- Weak-scaling normalized cost per core follows the expected ~1/log2(N) growth trend for FFT-based computations
- scaling_benchmark_results.csv contains one valid row per (problem, core count) pair with non-missing timings
- Dedalus test suite passes (`python3 -m dedalus test`) before benchmarking, ensuring the built extensions are correct
- Timings are monotonically decreasing (or plateauing) with increasing core count for the strong-scaling fixed-size problems

## Limitations

- Parallel efficiency degrades once the domain is decomposed to very few pencils per core; the >50% plateau is only claimed down to ~8 pencils per core
- Dedalus is documented as runnable 'easily on single AWS instances' — large multi-node weak-scaling runs need a cluster with MPI-parallel HDF5 and FFTW, and cloud single-instance results may not transfer
- Weak-scaling cost grows as 1/log2(N) rather than staying flat, so per-core cost never becomes perfectly constant for FFT-based spectral methods

## Evidence

- [methods] parallel efficiency threshold: [evidence span withheld — anchor status: not_found]
- [methods] FFTW MPI build: "--enable-shared --enable-mpi --enable-openmp --enable-threads"
- [methods] AWS usage: "Currently, dedalus can be run easily on single AWS instances."
- [methods] source install: "pip3 install -r requirements.txt"
- [abstract] parallelized solvers: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [readme] tooling: [evidence span withheld — anchor status: not_found]
