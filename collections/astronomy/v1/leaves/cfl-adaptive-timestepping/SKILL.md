---
name: cfl-adaptive-timestepping
description: Use when an explicit spectral PDE simulation (Dedalus, Fourier periodic
  domain) develops sharp gradients or shocks that would blow up a fixed timestep,
  and a CFL-restricted adaptive dt is needed — e.g., the Orszag-Tang vortex MHD test
  run to t=0.5.
when_to_use_negative:
- When solving steady-state or eigenvalue problems (e.g., `solver.solve_dense`/`solver.solve_sparse`
  subproblems as in the Mathieu or Rayleigh-Benard examples) — no timestep exists
  to restrict.
- When the flow has weak gradients such that a fixed timestep already satisfies the
  CFL condition and adaptive control only adds overhead.
- When shocks cannot be regularized by the explicit diffusion terms, since unregularized
  spectral expansions cannot represent discontinuities.
tools:
- name: Dedalus
  role: Framework that builds the compressible MHD IVP and provides the RK443 timestepper
    with CFL-restricted adaptive timestep; solvers are automatically parallelized
    with MPI.
  repo: https://github.com/DedalusProject/dedalus
- name: dedalus-feedstock
  role: Conda packaging of Dedalus for installation of the solver used for the adaptive-timestep
    run.
  repo: https://github.com/conda-forge/dedalus-feedstock
- name: numpy
  role: Array backend for field data and initial-condition construction in the simulation
    script.
- name: h5py
  role: Reads the HDF5 snapshot files written at each adaptively stepped checkpoint
    so the t=0.5 density field can be extracted.
  repo: https://github.com/h5py/h5py
- name: matplotlib
  role: Renders the extracted density field as a 2D map for visual verification of
    the shock structure.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:9f126f25005a86b4d139a08939e321d131ad74e765c425aa0516a722ba655c9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DedalusProject/dedalus
  tools:
  - Dedalus
  - MPI
  - Python
  - dedalus-feedstock
  - h5py
  - matplotlib
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# CFL-adaptive timestepping

## Summary

Configure an explicit IVP solve (e.g., Dedalus's RK443 timestepper) with a CFL-restricted adaptive timestep so that spectral simulations of high-Mach-number flows — like the 4096x4096-mode Orszag-Tang vortex with regularized shocks — remain stable up to the target time t=0.5.

## When to use

Use when an explicit spectral PDE simulation (Dedalus, Fourier periodic domain) develops sharp gradients or shocks that would blow up a fixed timestep, and a CFL-restricted adaptive dt is needed — e.g., the Orszag-Tang vortex MHD test run to t=0.5.

## When NOT to use

- When solving steady-state or eigenvalue problems (e.g., `solver.solve_dense`/`solver.solve_sparse` subproblems as in the Mathieu or Rayleigh-Benard examples) — no timestep exists to restrict.
- When the flow has weak gradients such that a fixed timestep already satisfies the CFL condition and adaptive control only adds overhead.
- When shocks cannot be regularized by the explicit diffusion terms, since unregularized spectral expansions cannot represent discontinuities.

## Inputs

- Compressible MHD problem specification on a periodic 2D Fourier domain (4096x4096 modes, 3/2 dealiasing)
- Orszag-Tang vortex initial conditions
- Explicit diffusion coefficients for momentum, heat, and magnetic field equations
- Target end time (t=0.5)

## Outputs

- Adaptive, CFL-restricted timestep sequence integrated to t=0.5
- HDF5 snapshot files written by the Dedalus IVP solver
- Density field extracted at t=0.5 and rendered as a 2D map via matplotlib.pyplot

## How to apply

Assemble the problem in Dedalus with `import dedalus.public as d3` on a periodic 2D Fourier domain with 4096x4096 modes and 3/2 dealiasing for the quadratic nonlinear terms; the dealiasing is required because the Orszag-Tang dynamics develop strong nonlinear couplings. Add the explicit diffusion terms for the momentum, heat, and magnetic field equations, which regularize the shocks that would otherwise break the spectral representation. Build the IVP solver with the RK443 timestepper and enable a CFL-restricted adaptive timestep, so dt shrinks as density and velocity extrema sharpen the shock fronts; the CFL condition on the fastest advective and Alfvén wave speeds sets the stability ceiling on dt at each step. Run the integration under MPI (Dedalus solvers are automatically parallelized) until t=0.5, writing HDF5 snapshots. Finally, load the snapshots with h5py, extract the density field, and confirm the timestep never exceeded the CFL limit and no numerical instability (e.g., NaN growth near shocks) occurred.

## Related tools

- **Dedalus** (Framework that builds the compressible MHD IVP and provides the RK443 timestepper with CFL-restricted adaptive timestep; solvers are automatically parallelized with MPI.) — https://github.com/DedalusProject/dedalus
- **dedalus-feedstock** (Conda packaging of Dedalus for installation of the solver used for the adaptive-timestep run.) — https://github.com/conda-forge/dedalus-feedstock
- **numpy** (Array backend for field data and initial-condition construction in the simulation script.)
- **h5py** (Reads the HDF5 snapshot files written at each adaptively stepped checkpoint so the t=0.5 density field can be extracted.) — https://github.com/h5py/h5py
- **matplotlib** (Renders the extracted density field as a 2D map for visual verification of the shock structure.)

## Examples

```
import numpy as np; import dedalus.public as d3; solver = problem.build_solver(d3.RK443)
```

## Evaluation signals

- Simulation reaches t=0.5 without NaNs or unbounded growth in the spectral coefficients near shock fronts.
- The adaptive timestep stays below the CFL restriction implied by the fastest advective/Alfvén speeds at every step.
- The density field at t=0.5 shows very sharp shock profiles rather than smeared or oscillatory fronts.
- Shock-shock interaction features appear in the upper-left and lower-right corners of the domain, matching the expected Orszag-Tang morphology.
- The 3/2 dealiasing of quadratic nonlinear terms is active, confirming aliasing errors are not contaminating the small-scale structure.

## Limitations

- Spectral methods with global bases cannot handle true discontinuities; stability relies on the explicit diffusion regularization of momentum, heat, and magnetic fields.
- The 4096x4096-mode grid at 3/2 dealiasing is computationally expensive and must be run under MPI parallelization.
- CFL adaptivity is only valid for the explicit IVP timesteppers (e.g., RK443); Dedalus's dense/sparse eigenvalue solvers do not use timestepping.

## Evidence

- [abstract] Dedalus capabilities span: "They also illustrate its capabilities with diverse examples, including optical network dynamics, magnetized shocks in plasmas, large-scale oceanic flows, low Reynolds number flows, stellar and"
- [intro] Python interface claim: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [readme] Framework claim from README: "Dedalus is a flexible framework for solving partial"
- [abstract] Abstract translation claim: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
