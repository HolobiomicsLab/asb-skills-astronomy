---
name: quasigeostrophic-flow-simulation
description: Use when reproducing or extending the 3D quasigeostrophic baroclinic
  instability example, or when studying the statistically saturated state of a sea
  of compact eddies that form, merge, and break up over several dynamical times.
when_to_use_negative:
- When the target is a linear/eigenvalue problem (e.g. Rayleigh-Benard growth rates
  or pipe flow eigenvalues) rather than nonlinear time integration of quasigeostrophic
  dynamics
- When the flow has not been integrated long enough to saturate — before t=200 the
  solution does not yet exhibit the compact-eddy statistically saturated state
- When working with non-spectral discretizations or non-Dedalus PDE frameworks, since
  the example relies on Dedalus's symbolic equation entry and compiled sparse solvers
edam_operation: http://edamontology.org/operation_3946
edam_topics: []
tools:
- name: Dedalus
  role: symbolic equation entry, spectral discretization, and MPI-parallel compiled
    sparse time integration of the quasigeostrophic equations
  repo: https://github.com/DedalusProject/dedalus
- name: Python
  role: driver language for the example script and post-processing
- name: numpy
  role: array manipulation of extracted final-time fields
- name: h5py
  role: reading the simulation's HDF5 output snapshots to extract w and p fields
  repo: https://github.com/h5py/h5py
- name: matplotlib
  role: rendering slice plots of potential vorticity and buoyancy perturbation fields
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:eee089c0ee65c7db8577221a3cfde2d31e08d45f336e99390d2383e767290f5a
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
  edam_operation: http://edamontology.org/operation_3946
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# quasigeostrophic-flow-simulation

## Summary

Reproduce the Dedalus 3D quasigeostrophic baroclinic instability example by spectrally evolving the potential vorticity (p) and buoyancy (w) fields on a 256x128x32-mode domain with Ekman pumping boundary conditions and depth-dependent stratification, integrating to t=200 to reach the statistically saturated state.

## When to use

Use when reproducing or extending the 3D quasigeostrophic baroclinic instability example, or when studying the statistically saturated state of a sea of compact eddies that form, merge, and break up over several dynamical times.

## When NOT to use

- When the target is a linear/eigenvalue problem (e.g. Rayleigh-Benard growth rates or pipe flow eigenvalues) rather than nonlinear time integration of quasigeostrophic dynamics
- When the flow has not been integrated long enough to saturate — before t=200 the solution does not yet exhibit the compact-eddy statistically saturated state
- When working with non-spectral discretizations or non-Dedalus PDE frameworks, since the example relies on Dedalus's symbolic equation entry and compiled sparse solvers

## Inputs

- Quasigeostrophic example script from dedalus/src/dedalus/examples
- Spectrally-discretized domain configuration (256x128x32 modes for w and p variables)
- Quasigeostrophic equations with Ekman pumping boundary conditions and depth-dependent stratification
- Simulation HDF5 snapshot output files

## Outputs

- MPI-parallel time-integrated solution reaching the statistically saturated state at t=200
- Final-time potential vorticity (p) and buoyancy perturbation (w) fields extracted from HDF5 snapshots
- Matplotlib slice plots showing compact eddies of the saturated state

## How to apply

Load the quasigeostrophic example script from dedalus/src/dedalus/examples and configure the spectrally-discretized domain with 256x128x32 modes for the buoyancy (w) and potential vorticity (p) variables. Set up the problem using Dedalus's symbolic vectorial equation entry, imposing Ekman pumping boundary conditions and depth-dependent stratification in the quasigeostrophic equations. Run the MPI-parallel time integration with Dedalus's compiled sparse solvers until t=200 so the flow reaches its statistically saturated state. Post-process the simulation's HDF5 output files with h5py and numpy to extract the final-time potential vorticity and buoyancy perturbation fields, then render slice plots with matplotlib to visualize the compact eddy structures.

## Related tools

- **Dedalus** (symbolic equation entry, spectral discretization, and MPI-parallel compiled sparse time integration of the quasigeostrophic equations) — https://github.com/DedalusProject/dedalus
- **Python** (driver language for the example script and post-processing)
- **numpy** (array manipulation of extracted final-time fields)
- **h5py** (reading the simulation's HDF5 output snapshots to extract w and p fields) — https://github.com/h5py/h5py
- **matplotlib** (rendering slice plots of potential vorticity and buoyancy perturbation fields)

## Examples

```
mpirun -n 4 python3 quasigeostrophic.py; import h5py; f = h5py.File('snapshots/snapshots_s1.h5', 'r'); p = f['tasks/p'][-1]
```

## Evaluation signals

- Integration runs to t=200 without failure
- HDF5 snapshot files exist at the final time and contain both the w (buoyancy) and p (potential vorticity) datasets
- Rendered slice plots show the compact eddies characteristic of the statistically saturated state (qualitative expert check against the reported example behavior)
- Domain configuration matches the documented 256x128x32 modal resolution for both variables

## Limitations

- The saturated-state structure is judged by qualitative expert check against the reported example behavior, not by a quantitative metric
- Dedalus writes HDF5 output, so parallel-HDF5/FFW build support (MPI-enabled HDF5 and FFTW) may be required for MPI-parallel runs
- Running to t=200 with 256x128x32 modes is computationally substantial; Dedalus currently runs easily on single AWS instances but larger parallel setups depend on the compiled sparse solver stack

## Evidence

- [other] saturated state resolution: [evidence span withheld — anchor status: not_found]
- [other] boundary conditions: [evidence span withheld — anchor status: not_found]
- [other] saturation finding: [evidence span withheld — anchor status: not_found]
- [intro] post-processing: "import h5py
import numpy as np"
- [abstract] framework claim: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods"
- [methods] hdf5 parallel build: "./configure --prefix=$BUILD_HOME CC=mpiicc CXX=mpiicpc F77=mpiifort \
                         --enable-shared --enable-parallel"
