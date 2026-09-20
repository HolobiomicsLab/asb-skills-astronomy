---
name: magnetohydrodynamic-simulation-setup
description: Use when a task requires solving high-Mach-number MHD flows that develop
  shocks with a spectral solver, specifically reproducing tests like the Orszag-Tang
  vortex (Figs. 11-12) up to a target simulation time such as t=0.5.
when_to_use_negative:
- The problem involves discontinuous shocks but you cannot add explicit diffusion
  or other regularization — global spectral methods need regularization to handle
  high-Mach shocks
- The domain geometry is not expressible as plain-text PDEs on Dedalus's supported
  spectral bases (e.g., unstructured mesh or embedded sharp material interfaces)
- The goal is a non-MHD PDE benchmark already covered by other Dedalus examples (e.g.,
  Rayleigh-Benard growth rates, Mathieu eigenvalues, pipe flow eigenvalues)
tools:
- name: Dedalus
  role: Spectral PDE framework used to specify the compressible MHD equations in plain
    text and run the parallelized Orszag-Tang simulation
  repo: https://github.com/DedalusProject/dedalus
- name: h5py
  role: Python wrapper used to load the HDF5 snapshot files written by Dedalus and
    extract the density field at t=0.5
  repo: https://github.com/h5py/h5py
- name: matplotlib
  role: Renders the extracted density field as a 2D map to confirm the sharp shock
    profiles
- name: numpy
  role: Array backend imported alongside dedalus.public as d3 for field and grid handling
- name: MPI
  role: Parallelization mechanism for running the 4096x4096-mode simulation; Dedalus
    solvers are automatically parallelized with MPI, and HDF5 must be built with parallel
    support
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:6e19b1bdc2e11d1b2a623558e4e4b38c2834bf4365cd5b83f4ba1bd693ddd650
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# magnetohydrodynamic-simulation-setup

## Summary

Set up and run compressible MHD simulations (e.g., the Orszag-Tang vortex test) with Dedalus's spectral methods, using explicit diffusion to regularize shocks and MPI-parallelized execution on a periodic 2D Fourier domain, then verify sharp shock structures in rendered density fields.

## When to use

Use when a task requires solving high-Mach-number MHD flows that develop shocks with a spectral solver, specifically reproducing tests like the Orszag-Tang vortex (Figs. 11-12) up to a target simulation time such as t=0.5.

## When NOT to use

- The problem involves discontinuous shocks but you cannot add explicit diffusion or other regularization — global spectral methods need regularization to handle high-Mach shocks
- The domain geometry is not expressible as plain-text PDEs on Dedalus's supported spectral bases (e.g., unstructured mesh or embedded sharp material interfaces)
- The goal is a non-MHD PDE benchmark already covered by other Dedalus examples (e.g., Rayleigh-Benard growth rates, Mathieu eigenvalues, pipe flow eigenvalues)

## Inputs

- Dedalus example Python script for the Orszag-Tang vortex (dedalus/src/dedalus/examples)
- Compressible MHD problem specification as plain-text vectorial equations on a periodic 2D Fourier domain (4096x4096 modes)
- Simulation parameters: 3/2 dealiasing factor, explicit diffusion coefficients, RK443 timestepper with CFL-restricted adaptive timestep, target time t=0.5

## Outputs

- HDF5 snapshot files of the MHD fields written during the run
- Density field extracted at t=0.5
- 2D matplotlib map of the density field showing sharp shocks and shock-shock interaction features in the upper-left and lower-right corners

## How to apply

Locate the Orszag-Tang example script among the example python scripts shipped in the Dedalus codebase (dedalus/src/dedalus/examples) and import numpy together with dedalus.public as d3. Assemble the compressible MHD problem on a periodic 2D Fourier domain discretized with 4096x4096 modes, applying 3/2 dealiasing to the quadratic nonlinear terms. Add explicit diffusion terms for the momentum, heat, and magnetic field equations — this regularization is what lets the global spectral method track sharp, high-Mach shocks without spurious oscillation — and configure the RK443 timestepper with a CFL-restricted adaptive timestep. Run under MPI (Dedalus solvers are automatically parallelized with MPI) until t=0.5, then load the written HDF5 snapshots with h5py and extract the density field for rendering.

## Related tools

- **Dedalus** (Spectral PDE framework used to specify the compressible MHD equations in plain text and run the parallelized Orszag-Tang simulation) — https://github.com/DedalusProject/dedalus
- **h5py** (Python wrapper used to load the HDF5 snapshot files written by Dedalus and extract the density field at t=0.5) — https://github.com/h5py/h5py
- **matplotlib** (Renders the extracted density field as a 2D map to confirm the sharp shock profiles)
- **numpy** (Array backend imported alongside dedalus.public as d3 for field and grid handling)
- **MPI** (Parallelization mechanism for running the 4096x4096-mode simulation; Dedalus solvers are automatically parallelized with MPI, and HDF5 must be built with parallel support)

## Examples

```
import numpy as np; import dedalus.public as d3
```

## Evaluation signals

- The rendered density field at t=0.5 resolves very sharp density shocks and shows shock-shock interaction features in the upper-left and lower-right corners of the domain (cf. Figs. 11-12)
- The Dedalus test suite passes before the run (python3 -m dedalus test)
- Quadratic nonlinear terms are dealiased at 3/2 on the 4096x4096 periodic Fourier grid
- The RK443 timestepper with CFL-restricted adaptive timestep advances stably to the target time t=0.5 without divergence
- HDF5 snapshot files written by the run load correctly with h5py and contain the density field at t=0.5

## Limitations

- Dedalus's global spectral method requires explicit diffusion (regularization) to handle high-Mach flows with shocks; without it, the spectral representation of discontinuities fails
- Large grids such as 4096x4096 require MPI parallelization; running on AWS, Dedalus is limited to running easily on single AWS instances, and HDF5/FFTW must be built with MPI support for parallel I/O
- Dedalus is GPL-3.0-or-later licensed, and no changelog is published in the article's discussion materials

## Evidence

- [abstract] Dedalus's spectral method handles shocks with regularization: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [abstract] MHD/shock examples are within Dedalus's demonstrated scope: "They also illustrate its capabilities with diverse examples, including optical network dynamics, magnetized shocks in plasmas, large-scale oceanic flows, low Reynolds number flows, stellar and"
- [intro] Dedalus is a flexible framework for solving partial differential equations using modern spectral methods.: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [intro] import numpy as np
import dedalus.public as d3: "import numpy as np
import dedalus.public as d3"
- [methods] Parallel HDF5 build requirement for MPI runs: "./configure --prefix=$BUILD_HOME CC=mpiicc CXX=mpiicpc F77=mpiifort \
                         --enable-shared --enable-parallel"
- [methods] Test suite validation before simulation runs: "Ensure tests pass by running:
  sh
  python3 -m dedalus test
  "
- [readme] h5py role for loading HDF5 snapshots: [evidence span withheld — anchor status: external_doc]
- [readme] Dedalus package description: [evidence span withheld — anchor status: external_doc]
