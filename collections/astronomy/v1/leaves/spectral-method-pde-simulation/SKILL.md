---
name: spectral-method-pde-simulation
description: Use when a PDE-based physical simulation (fluid dynamics, waves, eigenvalue
  problems) must be solved at high accuracy with spectral methods, e.g. reproducing
  resolution-comparison benchmarks or spectral-convergence series.
when_to_use_negative:
- The problem needs only finite-volume or finite-difference accuracy and a code like
  Athena is already available — this skill targets the spectral-method route and its
  convergence properties, not generic CFD.
- The equations cannot be expressed in Dedalus's symbolic vectorial form or the domain/basis
  geometry does not match Dedalus's supported bases.
- You need validated off-the-shelf solver binaries without installing dependencies
  — Dedalus requires source builds (numpy/FFTW/HDF5/OpenBLAS) or the conda-forge feedstock.
edam_operation: http://edamontology.org/operation_0438
tools:
- name: Dedalus
  role: Core spectral PDE solver; translates plain-text equations into parallelized
    solvers and provides example benchmark scripts
  repo: https://github.com/DedalusProject/dedalus
- name: Python
  role: Primary language in which Dedalus is written and in which benchmark scripts
    are run
- name: numpy
  role: Array/numpy-based specification of domains, initial conditions, and analytic
    comparisons (e.g. true_evals = (n * np.pi / Lx)**2)
- name: h5py
  role: Reading/writing late-time snapshot data saved through HDF5 analysis handlers
  repo: https://github.com/h5py/h5py
- name: matplotlib
  role: Rendering side-by-side resolution-comparison figures and result plots
- name: scipy
  role: Special functions (e.g. scipy.special.jv) used in benchmark problem setup
  repo: https://github.com/scipy/scipy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:d9e048a8e46dc29cb2924a19b672d16654d19021c798e5fda5ed5208a2816853
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
  - scipy
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# spectral-method-pde-simulation

## Summary

Simulate partial differential equations with Dedalus, a Python framework that translates plain-text vectorial equations into parallelized global spectral-method solvers, as used in benchmarks such as the Kelvin-Helmholtz instability comparison (2048^2 spectral modes vs 16384^2 finite-volume cells).

## When to use

Use when a PDE-based physical simulation (fluid dynamics, waves, eigenvalue problems) must be solved at high accuracy with spectral methods, e.g. reproducing resolution-comparison benchmarks or spectral-convergence series.

## When NOT to use

- The problem needs only finite-volume or finite-difference accuracy and a code like Athena is already available — this skill targets the spectral-method route and its convergence properties, not generic CFD.
- The equations cannot be expressed in Dedalus's symbolic vectorial form or the domain/basis geometry does not match Dedalus's supported bases.
- You need validated off-the-shelf solver binaries without installing dependencies — Dedalus requires source builds (numpy/FFTW/HDF5/OpenBLAS) or the conda-forge feedstock.

## Inputs

- Plain-text symbolic PDE specification (vectorial equations) entered via the Dedalus d3 interface
- 2D shear-flow domain configuration with initial conditions and timestep settings for benchmarks
- Resolution ladder specification (e.g. coarse grids up to 2048^2 Fourier modes)
- Reference field data from an independent solver (e.g. finite-volume Athena at 16384^2 cells)

## Outputs

- Late-time simulation snapshots stored via HDF5 analysis handlers
- Spectral-convergence series across increasing resolutions
- Field-level agreement metrics (relative differences in vorticity/density fields)
- Eigenvalue/growth-rate results (e.g. Rayleigh-Benard growth rates at Rayleigh = 1710, Prandtl = 1; Mathieu eigenvalues over a range of q values)

## How to apply

Write the governing equations symbolically using Dedalus's `dedalus.public as d3` interface, select appropriate bases (e.g. Fourier modes for 2D shear-flow domains), and configure timestepping with matched initial conditions and timestep settings. Run the simulation at a ladder of increasing resolutions — for the Kelvin-Helmholtz benchmark, coarse grids up to 2048^2 Fourier modes — saving late-time snapshots through HDF5 analysis handlers (`h5py`). For eigenvalue-type subproblems, choose between `solver.solve_dense(solver.subproblems[0], rebuild_matrices=True)` for small dense solves and `solver.solve_sparse(solver.subproblems[1], NEV, target=target)` for sparse eigenvalues over a parameter range. Validate the setup by running the built-in test suite (`python3 -m dedalus test`) before production runs. Judge success by spectral convergence across the resolution ladder and by field-level agreement (relative differences in vorticity/density fields) against a reference solution aligned at the same physical time.

## Related tools

- **Dedalus** (Core spectral PDE solver; translates plain-text equations into parallelized solvers and provides example benchmark scripts) — https://github.com/DedalusProject/dedalus
- **Python** (Primary language in which Dedalus is written and in which benchmark scripts are run)
- **numpy** (Array/numpy-based specification of domains, initial conditions, and analytic comparisons (e.g. true_evals = (n * np.pi / Lx)**2))
- **h5py** (Reading/writing late-time snapshot data saved through HDF5 analysis handlers) — https://github.com/h5py/h5py
- **matplotlib** (Rendering side-by-side resolution-comparison figures and result plots)
- **scipy** (Special functions (e.g. scipy.special.jv) used in benchmark problem setup) — https://github.com/scipy/scipy

## Examples

```
import numpy as np
import dedalus.public as d3
```

## Evaluation signals

- Run `python3 -m dedalus test` and confirm all built-in tests pass before production runs.
- Waves-on-a-string eigenvalues must match analytic values (n * np.pi / Lx)**2 via relative error.
- Rayleigh-Benard growth-rate curves (Rayleigh = 1710, Prandtl = 1) reproduce as sparse eigenvalue solves converge over the horizontal-wavenumber range.
- The 2048^2-mode spectral late-time state is quantitatively comparable to the 16384^2-cell finite-volume reference (relative differences in vorticity/density fields), with no spurious secondary instabilities in the vortex rolls.
- HDF5 snapshot files are readable via h5py and snapshots from different resolutions align at the same physical time.

## Limitations

- Building Dedalus from source requires careful dependency configuration (numpy against MKL, HDF5 with parallel MPI support via --enable-parallel, FFTW with --enable-mpi --enable-openmp --enable-threads).
- High-resolution runs (e.g. 2048^2 modes, or matching a 16384^2-cell finite-volume reference) are computationally expensive; Dedalus 'can be run easily on single AWS instances', implying larger scales need more resources.
- Accuracy guarantees are spectral-method-specific: coarse finite-volume runs can produce unphysical secondary instabilities in vortex rolls that spectral runs avoid, so cross-code comparisons must match physical times and diagnostics carefully.

## Evidence

- [abstract] Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods.: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [intro] Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification.: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [methods] Ensure tests pass by running: python3 -m dedalus test: "Ensure tests pass by running:
  sh
  python3 -m dedalus test"
- [methods] Code repository listed in methods: "* Code repository: https://github.com/DedalusProject/dedalus"
- [readme] Feedstock summary of Dedalus package: [evidence span withheld — anchor status: external_doc]
- [methods] Currently, dedalus can be run easily on single AWS instances.: "Currently, dedalus can be run easily on single AWS instances."
