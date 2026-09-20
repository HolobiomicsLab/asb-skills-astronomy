---
name: spectral-pde-solver-configuration
description: Use when a PDE initial/boundary-value or eigenvalue problem must be solved
  with spectral accuracy in Dedalus, e.g. symbolic vectorial equation entry, 2D/3D
  Fourier-Chebyshev domains, sparse/dense eigen-solves, or shock/eddy-resolving IVPs.
when_to_use_negative:
- Problems with geometries or boundary conditions not representable by Dedalus's global
  spectral bases (e.g. unstructured meshes or discontinuous material interfaces)
- Strongly shock-dominated problems solved without the explicit diffusion regularization
  the paper uses for the Orszag-Tang MHD test
- Tasks that only need HDF5 post-processing of existing simulation output rather than
  setting up and running a new spectral solve
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Dedalus
  role: Framework that translates plain-text equations into efficient and parallelized
    global spectral solvers; provides symbolic equation entry, sparse/dense solvers,
    and HDF5 output
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Array backend and initial-condition construction for the spectrally discretized
    fields
- name: h5py
  role: Reading/writing the HDF5 snapshot files produced by the time integration
  repo: https://github.com/h5py/h5py
- name: matplotlib
  role: Rendering slice/2D maps of solution fields (e.g. density shocks, eddy structures)
    for validation
- name: OpenBLAS
  role: Optimized BLAS/LAPACK library that underlies the dense/sparse linear algebra
    in the compiled solvers
  repo: https://github.com/OpenMathLib/OpenBLAS
- name: scipy
  role: Supporting scientific functions (e.g. scipy.special.jv) used alongside Dedalus
    scripts
  repo: https://github.com/scipy/scipy
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:02f6bc730880f3f6fee9a989da1e1084edce667e8920d1fc85ea39b8f7b0be61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DedalusProject/dedalus
  tools:
  - Dedalus
  - MPI
  - OpenBLAS
  - Python
  - h5py
  - matplotlib
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# spectral-pde-solver-configuration

## Summary

Configure Dedalus's global spectral method to translate plain-text, symbolically specified PDEs into efficient, MPI-parallelized solvers, selecting sparse or dense solves, dealiasing, timesteppers, and basis discretizations appropriate to the problem.

## When to use

Use when a PDE initial/boundary-value or eigenvalue problem must be solved with spectral accuracy in Dedalus, e.g. symbolic vectorial equation entry, 2D/3D Fourier-Chebyshev domains, sparse/dense eigen-solves, or shock/eddy-resolving IVPs.

## When NOT to use

- Problems with geometries or boundary conditions not representable by Dedalus's global spectral bases (e.g. unstructured meshes or discontinuous material interfaces)
- Strongly shock-dominated problems solved without the explicit diffusion regularization the paper uses for the Orszag-Tang MHD test
- Tasks that only need HDF5 post-processing of existing simulation output rather than setting up and running a new spectral solve

## Inputs

- Plain-text/symbolic PDE specification (vectorial equations, boundary conditions, parameters) entered via dedalus.public as d3
- Domain/basis configuration: mode counts (e.g. 4096x4096 or 256x128x32), dealiasing factor (3/2), periodic or Chebyshev coordinates
- Numerical parameters: timestepper choice (RK443), CFL-based adaptive timestep, explicit diffusion coefficients, target simulation time or eigenvalue target

## Outputs

- MPI-parallel time-integrated solution fields written as HDF5 snapshot files (e.g. density at t=0.5; potential vorticity and buoyancy perturbation at t=200)
- Eigenvalues/growth rates from sparse or dense subproblem solves (e.g. Rayleigh-Benard growth rates, pipe-flow eigenvalues, waves-on-a-string eigenvalues)

## How to apply

Write the governing equations with Dedalus's symbolic vectorial interface (import dedalus.public as d3 with numpy), discretizing the domain with the appropriate basis combination and mode counts — e.g. a periodic 2D Fourier domain with 4096x4096 modes and 3/2 dealiasing for quadratic nonlinear terms, or a 256x128x32 spectrally-discretized domain for the 3D quasigeostrophic w (buoyancy) and p (potential vorticity) variables. Impose the required boundary conditions symbolically (e.g. Ekman pumping, depth-dependent stratification) and add explicit diffusion for high-Mach/shock problems, with an RK443 timestepper and CFL-restricted adaptive timestep. Choose the solver mode by problem type: solver.solve_sparse for eigenvalue problems (Rayleigh-Benard growth rates at Rayleigh = 1710, Prandtl = 1) or solver.solve_dense (Mathieu problem, rebuild_matrices=True) when the full spectrum is needed. Run under MPI — Dedalus solvers are automatically parallelized — integrate to the target time (e.g. t=0.5 for Orszag-Tang, t=200 for quasigeostrophic saturation), and write HDF5 snapshots.

## Related tools

- **Dedalus** (Framework that translates plain-text equations into efficient and parallelized global spectral solvers; provides symbolic equation entry, sparse/dense solvers, and HDF5 output) — https://github.com/DedalusProject/dedalus
- **numpy** (Array backend and initial-condition construction for the spectrally discretized fields)
- **h5py** (Reading/writing the HDF5 snapshot files produced by the time integration) — https://github.com/h5py/h5py
- **matplotlib** (Rendering slice/2D maps of solution fields (e.g. density shocks, eddy structures) for validation)
- **OpenBLAS** (Optimized BLAS/LAPACK library that underlies the dense/sparse linear algebra in the compiled solvers) — https://github.com/OpenMathLib/OpenBLAS
- **scipy** (Supporting scientific functions (e.g. scipy.special.jv) used alongside Dedalus scripts) — https://github.com/scipy/scipy

## Examples

```
import numpy as np; import dedalus.public as d3; solver.solve_sparse(solver.subproblems[1], NEV, target=target)
```

## Evaluation signals

- Integration reaches the target time without failure (t=0.5 for Orszag-Tang; t=200 for the statistically saturated quasigeostrophic state) and HDF5 snapshots exist containing the expected fields (w, p, or density)
- Eigenvalue verification: waves-on-a-string eigenvalues match (n * np.pi / Lx)**2 by relative error, and Rayleigh-Benard growth rates computed at Rayleigh = 1710, Prandtl = 1 are recovered from solver.solve_sparse
- Rendered fields show the expected physics: sharp density shocks with shock-shock interaction features in the upper-left and lower-right corners (Orszag-Tang, t=0.5), or compact eddies forming, merging, and breaking up (quasigeostrophic, t=200)
- Quadratic nonlinear terms are dealiased at 3/2 on the configured grid so spectral convergence is not degraded
- Dedalus test suite passes after installation (python3 -m dedalus test) before configuring production runs

## Limitations

- High-Mach-number flows with shocks require explicit diffusion regularization; unregularized spectral methods fail on discontinuities
- Dedalus is developed by researchers studying astrophysical, geophysical, and biological fluid dynamics — problems far outside these PDE classes may need different frameworks
- Building from source requires assembling compatible parallel libraries (HDF5 with MPI support, FFTW with --enable-mpi, MKL-built numpy), which is error-prone; conda-forge provides prebuilt packages
- Saturated turbulence statistics are qualitative: the t=200 quasigeostrophic eddy state was validated by qualitative expert check against reported example behavior, not a quantitative metric

## Evidence

- [intro] Symbolic equation entry: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [abstract] Translated parallel solvers: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [results] Sparse eigen-solve usage: "solver.solve_sparse(solver.subproblems[1], NEV, target=target)"
- [results] Sparse solve for growth rates: [evidence span withheld — anchor status: not_found]
- [results] Eigenvalue correctness check: "true_evals = (n * np.pi / Lx)**2
relative_error = np.abs(evals - true_evals) / true_evals"
- [methods] Install then test: "pip3 install -r requirements.txt
     python3 setup.py build_ext --inplace"
- [readme] README scope statement: "Dedalus is a flexible framework for solving partial"
- [methods] Test suite invocation: "python3 -m dedalus test"
