---
name: spectral-ivp-benchmark-execution
description: 'Use when reproducing or extending parallel scaling studies of spectral
  PDE solvers: you need to run hydro/MHD IVP benchmarks across multiple core counts
  and compute parallel efficiency from wall-clock timings.'
when_to_use_negative:
- The problem is a sparse/dense eigenvalue or linear-stability analysis (e.g. Rayleigh-Benard
  growth rates, pipe-flow eigenvalues) rather than a time-stepping IVP scaling benchmark
- The target code is not a global spectral-method PDE solver with pencil decomposition;
  FFT-based 1/log2(N) weak-scaling expectations do not transfer to non-spectral codes
- Only single-node serial execution is available and no MPI core-count ladder can
  be run, since strong/weak scaling ratios cannot be formed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_2269
tools:
- name: Dedalus
  role: Spectral PDE framework whose MPI-parallelized IVP solvers are benchmarked;
    pencil-decomposed execution across core counts
  repo: https://github.com/DedalusProject/dedalus
- name: h5py
  role: Python interface to parallel HDF5 output used by Dedalus IVP runs
  repo: https://github.com/h5py/h5py
- name: numpy
  role: Arrays and arithmetic for computing strong-scaling efficiency and weak-scaling
    cost from timings
- name: matplotlib
  role: Plotting scaling curves from scaling_benchmark_results.csv
- name: OpenBLAS
  role: Optimized BLAS backend built from source as a dependency for the spectral
    solver's dense linear algebra
  repo: https://github.com/OpenMathLib/OpenBLAS
- name: Python
  role: Primary language in which Dedalus and the benchmark scripts are written and
    executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:c6ecf036f4348c9f9c83412f179fbb5f9b68568cba5af31e55081d4fee663e3d
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# spectral-ivp-benchmark-execution

## Summary

Executes Dedalus's MPI-parallel initial value problem (IVP) benchmarks for 3D Boussinesq hydrodynamic and magnetohydrodynamic problems in Fourier-Fourier-Chebyshev domains, recording wall-clock timings to quantify strong- and weak-scaling behavior of the FFT-based pencil-decomposed solver.

## When to use

Use when reproducing or extending parallel scaling studies of spectral PDE solvers: you need to run hydro/MHD IVP benchmarks across multiple core counts and compute parallel efficiency from wall-clock timings.

## When NOT to use

- The problem is a sparse/dense eigenvalue or linear-stability analysis (e.g. Rayleigh-Benard growth rates, pipe-flow eigenvalues) rather than a time-stepping IVP scaling benchmark
- The target code is not a global spectral-method PDE solver with pencil decomposition; FFT-based 1/log2(N) weak-scaling expectations do not transfer to non-spectral codes
- Only single-node serial execution is available and no MPI core-count ladder can be run, since strong/weak scaling ratios cannot be formed

## Inputs

- Dedalus repository checkout (github.com/DedalusProject/dedalus) with preinstalled example/benchmark scripts
- MPI-parallel HDF5 and FFTW builds (CC=mpiicc, --enable-parallel; --enable-mpi --enable-threads)
- Cluster, cloud, or AWS EC2 allocation with a chosen ladder of core counts
- Hydrodynamic and MHD 3D Boussinesq IVP benchmark scripts

## Outputs

- Per-run wall-clock timings for each (problem, core count) configuration
- Strong-scaling parallel efficiency values (baseline time / n-core time x n)
- Weak-scaling normalized cost per core count
- scaling_benchmark_results.csv with one row per (problem, core count) configuration

## How to apply

Build Dedalus from the deposited repository (pip3 install -r requirements.txt; python3 setup.py build_ext --inplace) against MPI-enabled HDF5 (--enable-parallel) and FFTW (--enable-mpi --enable-threads), then verify the install with python3 -m dedalus test. Launch MPI executions of the hydrodynamic and MHD IVP scaling benchmarks (corresponding to Fig. 8 of the paper) across a ladder of core counts on a cluster or AWS EC2 instance, recording per-run wall-clock time. Compute strong-scaling parallel efficiency as (single-node baseline time / n-core time) x n and check that efficiency stays above ~50% down to roughly 8 pencils per core. Compute weak-scaling normalized cost per core count and verify the expected ~1/log2(N) growth, which follows from the log-scaling of FFT-based transposes in Dedalus's global spectral method. Assemble one row per (problem, core count) into scaling_benchmark_results.csv for downstream validation.

## Related tools

- **Dedalus** (Spectral PDE framework whose MPI-parallelized IVP solvers are benchmarked; pencil-decomposed execution across core counts) — https://github.com/DedalusProject/dedalus
- **h5py** (Python interface to parallel HDF5 output used by Dedalus IVP runs) — https://github.com/h5py/h5py
- **numpy** (Arrays and arithmetic for computing strong-scaling efficiency and weak-scaling cost from timings)
- **matplotlib** (Plotting scaling curves from scaling_benchmark_results.csv)
- **OpenBLAS** (Optimized BLAS backend built from source as a dependency for the spectral solver's dense linear algebra) — https://github.com/OpenMathLib/OpenBLAS
- **Python** (Primary language in which Dedalus and the benchmark scripts are written and executed)

## Examples

```
pip3 install -r requirements.txt; python3 setup.py build_ext --inplace && python3 -m dedalus test && mpirun -np 64 python3 examples/boussinesq/ivp/boussinesq_mhd.py
```

## Evaluation signals

- Strong-scaling parallel efficiency (T1 / Tn x n) remains above ~50% through the ~8-pencils-per-core regime for both hydro and MHD problems
- Weak-scaling normalized cost grows approximately as 1/log2(N) across core counts, as expected for FFT-based computations
- scaling_benchmark_results.csv contains one row per (problem, core count) configuration with no missing timings
- python3 -m dedalus test passes before benchmarking, confirming the MPI-enabled HDF5/FFTW build is correct

## Limitations

- Dedalus's efficient scaling to thousands of cores requires large 3D problem sizes; small problems exhaust parallelism and efficiency degrades below ~8 pencils per core
- The ~1/log2(N) weak-scaling trend is specific to FFT-based global spectral methods and pencil decomposition, not a general parallel-computing law
- Building the full dependency stack (parallel HDF5, MPI-enabled FFTW, MKL/OpenBLAS-backed numpy) is required before benchmarks run; pre-built serial installs will not exercise the MPI path

## Evidence

- [other] weak scaling trend: [evidence span withheld — anchor status: not_found]
- [other] efficiency plateau: [evidence span withheld — anchor status: not_found]
- [methods] install from source: "pip3 install -r requirements.txt
     python3 setup.py build_ext --inplace"
- [methods] test suite gate: "Ensure tests pass by running:
  sh
  python3 -m dedalus test"
- [methods] parallel HDF5 build: "./configure --prefix=$BUILD_HOME CC=mpiicc CXX=mpiicpc F77=mpiifort \
                         --enable-shared --enable-parallel"
- [methods] FFTW MPI build: "--enable-shared \
                         --enable-mpi --enable-openmp --enable-threads"
- [abstract] solver translation: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
