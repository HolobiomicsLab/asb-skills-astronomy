---
name: weak-scaling-efficiency-evaluation
description: Use when benchmarking an MPI-parallel spectral/FFT solver at fixed work
  per core and you need to quantify normalized weak-scaling cost across core counts
  and compare it to a 1/log2(N) reference trend.
when_to_use_negative:
- You are studying strong scaling (fixed problem size, varying cores) rather than
  fixed work per core — use a strong-scaling efficiency metric instead.
- The solver does not rely on global FFT-based transforms with all-to-all transposes,
  so the 1/log2(N) expectation does not apply as the acceptance criterion.
- Problem sizes cannot be co-scaled with core counts (e.g., fixed domain resolution
  required), making a weak-scaling design infeasible.
tools:
- name: Dedalus
  role: Spectral PDE framework providing the 3D Boussinesq hydro/MHD IVP scaling benchmarks
    whose MPI weak-scaling behavior is measured
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Numerical computation and fitting of the measured weak-scaling costs against
    the 1/log2(N) reference curve
- name: matplotlib
  role: Plotting weak-scaling cost versus core count for comparison with the expected
    trend
- name: h5py
  role: Handles parallel HDF5 output for benchmark runs on MPI-enabled builds
  repo: https://github.com/h5py/h5py
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:89efda9feae8c085c216073098fe71c58c507e7241f980c1d077f1771ef352a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DedalusProject/dedalus
  tools:
  - Dedalus
  - Python
  - h5py
  - matplotlib
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# weak-scaling-efficiency-evaluation

## Summary

Evaluates how parallel cost per core grows as problem size is scaled with core count for Dedalus's 3D Boussinesq hydro/MHD spectral benchmarks, verifying the theoretically expected ~1/log2(N) growth for FFT-based transforms.

## When to use

Use when benchmarking an MPI-parallel spectral/FFT solver at fixed work per core and you need to quantify normalized weak-scaling cost across core counts and compare it to a 1/log2(N) reference trend.

## When NOT to use

- You are studying strong scaling (fixed problem size, varying cores) rather than fixed work per core — use a strong-scaling efficiency metric instead.
- The solver does not rely on global FFT-based transforms with all-to-all transposes, so the 1/log2(N) expectation does not apply as the acceptance criterion.
- Problem sizes cannot be co-scaled with core counts (e.g., fixed domain resolution required), making a weak-scaling design infeasible.

## Inputs

- Dedalus 3D Boussinesq hydrodynamic and MHD IVP scaling benchmark scripts from the deposited repository
- MPI-enabled Dedalus build with parallel HDF5 and MPI-FFTW support
- Chosen set of core counts (with problem size held proportional to core count)
- Cluster scheduler or AWS EC2 instance access

## Outputs

- Per-core-count wall-clock timings from MPI runs
- Normalized weak-scaling cost values per core count
- scaling_benchmark_results.csv with one row per (problem, core count) configuration
- Assessment of whether weak-scaling cost follows the ~1/log2(N) trend

## How to apply

Set up the Dedalus 3D Boussinesq hydrodynamic and MHD IVP benchmarks in a Fourier-Fourier-Chebyshev domain, choosing problem sizes so the work per core (pencils per core) stays constant as core count increases. Launch MPI-parallel runs on a cluster or AWS EC2 instance built with MPI-enabled parallel HDF5 and FFTW (e.g., HDF5 configured with --enable-parallel and FFTW with --enable-mpi), recording per-run wall-clock timings for each core count. For each core count N, compute the normalized weak-scaling cost (cost per core, typically wall-time times N divided by the work unit) and fit/inspect the trend against the theoretically expected ~1/log2(N) growth arising from FFT-based global transposes. Assemble timings and weak-scaling costs into scaling_benchmark_results.csv with one row per (problem, core count) configuration. Accept the result if weak-scaling costs follow the ~1/log2(N) trend reported for the suite; also cross-check strong-scaling efficiency (>50% down to ~8 pencils per core) on the same runs.

## Related tools

- **Dedalus** (Spectral PDE framework providing the 3D Boussinesq hydro/MHD IVP scaling benchmarks whose MPI weak-scaling behavior is measured) — https://github.com/DedalusProject/dedalus
- **numpy** (Numerical computation and fitting of the measured weak-scaling costs against the 1/log2(N) reference curve)
- **matplotlib** (Plotting weak-scaling cost versus core count for comparison with the expected trend)
- **h5py** (Handles parallel HDF5 output for benchmark runs on MPI-enabled builds) — https://github.com/h5py/h5py

## Examples

```
pip3 install -r requirements.txt; python3 setup.py build_ext --inplace; mpirun -np N python3 benchmarks/boussinesq_mhd_scaling.py
```

## Evaluation signals

- Weak-scaling normalized cost per core grows approximately as 1/log2(N) across the tested core counts
- scaling_benchmark_results.csv contains one row per (problem, core count) configuration with recorded timings
- Runs execute on an MPI build with parallel HDF5 (--enable-parallel) and MPI-FFTW (--enable-mpi) support
- Companion strong-scaling efficiencies on the same benchmarks stay above ~50% down to ~8 pencils per core
- Dedalus test suite passes before benchmarking (python3 -m dedalus test)

## Limitations

- Weak-scaling efficiency is bounded by the 1/log2(N) growth of FFT-based global transposes, so ideal flat scaling is not expected even for a well-parallelized FFT code
- Dedalus was noted to run easily only on single AWS instances, so very large core counts require a proper cluster with MPI-I/O support
- Requires per-core work (pencils per core) to be held constant; deviations from that design confound the weak-scaling interpretation

## Evidence

- [results] Weak-scaling trend: [evidence span withheld — anchor status: not_found]
- [results] Strong-scaling plateau: [evidence span withheld — anchor status: not_found]
- [results] Benchmark scope: [evidence span withheld — anchor status: not_found]
- [methods] Build prerequisites: "./configure --prefix=$BUILD_HOME CC=mpiicc CXX=mpiicpc F77=mpiifort \
                         --enable-shared --enable-parallel"
- [methods] Install procedure: "pip3 install -r requirements.txt
     python3 setup.py build_ext --inplace"
