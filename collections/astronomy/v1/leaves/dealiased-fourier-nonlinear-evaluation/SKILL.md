---
name: dealiased-fourier-nonlinear-evaluation
description: Use when solving PDEs with Dedalus on a periodic 2D Fourier domain where
  quadratic nonlinear products of fields (e.g. convective and Lorentz-force terms
  in compressible MHD) must be evaluated on a mode grid such as 4096x4096.
when_to_use_negative:
- The problem is linear or contains no quadratic nonlinear products — the 3/2 truncation
  buys nothing and only wastes the extra grid points.
- The flow is high-Mach-number/shock-forming but no explicit diffusion or other regularization
  is planned — bare spectral evaluation will produce Gibbs ringing rather than resolved
  shocks.
- The domain is not periodic or the variables are not naturally represented in a Fourier
  basis — a different Dedalus basis (e.g. Chebyshev/Zernuke-style) than the 2D periodic
  Fourier setup is required.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: Dedalus
  role: Spectral PDE framework that translates plain-text equations into parallelized
    solvers and applies 3/2 dealiasing to quadratic nonlinear terms on the Fourier
    grid
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Array backend for the mode-coefficient fields and dealiased spectral transforms
- name: h5py
  role: Reads the HDF5 snapshot files written by the solver to verify the evolved
    fields
  repo: https://github.com/h5py/h5py
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:998ff3431fcca2ce05d520ca5dc64ba13210fe63723d8175b4b5e7f496479666
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DedalusProject/dedalus
  tools:
  - Dedalus
  - MPI
  - Python
  - h5py
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# dealiased-fourier-nonlinear-evaluation

## Summary

Evaluate quadratic nonlinear terms on a spectral Fourier grid with 3/2 (two-thirds) dealiasing so that aliasing errors from products of modes are removed before each timestep of a Dedalus PDE solve. This is what lets Dedalus resolve sharp spectral features, e.g. the density shocks of the Orszag-Tang vortex, without Gibbs-contaminated nonlinear terms.

## When to use

Use when solving PDEs with Dedalus on a periodic 2D Fourier domain where quadratic nonlinear products of fields (e.g. convective and Lorentz-force terms in compressible MHD) must be evaluated on a mode grid such as 4096x4096.

## When NOT to use

- The problem is linear or contains no quadratic nonlinear products — the 3/2 truncation buys nothing and only wastes the extra grid points.
- The flow is high-Mach-number/shock-forming but no explicit diffusion or other regularization is planned — bare spectral evaluation will produce Gibbs ringing rather than resolved shocks.
- The domain is not periodic or the variables are not naturally represented in a Fourier basis — a different Dedalus basis (e.g. Chebyshev/Zernuke-style) than the 2D periodic Fourier setup is required.

## Inputs

- Plain-text/symbolic PDE equations (e.g. compressible MHD: momentum, heat/continuity, induction) entered via the Dedalus d3 public interface
- 2D periodic Fourier domain specification (Lx, Ly and 4096x4096 mode count, with 3/2 dealiasing for quadratic nonlinear terms)
- Explicit diffusion coefficients for momentum, heat, and magnetic field equations
- Timestepper configuration (RK443) with CFL-restricted adaptive timestep

## Outputs

- Dealiased spectral coefficient fields for each evolved variable at every timestep
- HDF5 snapshot files (.h5) containing the simulation state (e.g. density at t=0.5), readable with h5py
- 2D rendered field maps (matplotlib.pyplot) showing shock profiles

## How to apply

Declare the problem with Dedalus's symbolic vectorial equation interface (d3) after importing numpy, specifying the physics equations in plain text; Dedalus translates these into spectral solvers. Discretize the periodic 2D domain with a Fourier basis at the target resolution (e.g. 4096x4096 modes) and enable 3/2 dealiasing so quadratic nonlinear terms are evaluated on a coarsened one-and-a-half-times grid before truncation, eliminating aliasing of products of modes. Add explicit diffusion terms to the momentum, heat, and magnetic field equations to regularize shocks in high-Mach-number flows, then run with an explicit timestepper (RK443) under an CFL-restricted adaptive timestep, parallelized via MPI as Dedalus solvers are automatically. Snapshots are written as HDF5, which can be verified with h5py; the payoff is sharp shock profiles like the Orszag-Tang density shocks at t=0.5 rather than ringing artifacts.

## Related tools

- **Dedalus** (Spectral PDE framework that translates plain-text equations into parallelized solvers and applies 3/2 dealiasing to quadratic nonlinear terms on the Fourier grid) — https://github.com/DedalusProject/dedalus
- **numpy** (Array backend for the mode-coefficient fields and dealiased spectral transforms)
- **h5py** (Reads the HDF5 snapshot files written by the solver to verify the evolved fields) — https://github.com/h5py/h5py

## Examples

```
import numpy as np; import dedalus.public as d3
```

## Evaluation signals

- Density field at t=0.5 of the Orszag-Tang run shows very sharp shock profiles with no large-scale ringing from the nonlinear products.
- Shock-shock interaction features appear in the upper-left and lower-right corners of the 2D domain, matching the expected pattern.
- Simulation remains stable to t=0.5 under the CFL-restricted adaptive timestep with the RK443 timestepper (no blow-up from aliasing).
- Dealiasing is set to 3/2 for all quadratic nonlinear terms in the problem specification; coarse-grid (dealiased) evaluation can be confirmed from the domain/basis configuration.
- HDF5 snapshots open cleanly with h5py and contain the density field at the target simulation time.

## Limitations

- Dedalus solves PDEs with global spectral methods, so genuinely discontinuous solutions (shocks) require explicit diffusion regularization; without it, spectral convergence breaks down near shocks.
- Quadratic nonlinear terms only — higher-order nonlinearity needs a different dealiasing rule than the 3/2 truncation.
- Running at 4096x4096 modes requires MPI parallelization and suitable linear-algebra libraries (OpenBLAS, FFTW built with MPI/OpenMP threads); the paper notes Dedalus runs most easily on single AWS instances.
- h5py requires Python 3.10+ (or 3.11+ per the h5py/h5py README).

## Evidence

- [intro] Dedalus spectral framework with dealiasing: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [intro] Import pattern for a Dedalus problem: "import numpy as np
import dedalus.public as d3"
- [abstract] Spectral solver translation: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [abstract] Shock example among Dedalus capabilities: "magnetized shocks in plasmas"
- [readme] Package summary from conda-forge feedstock: [evidence span withheld — anchor status: external_doc]
