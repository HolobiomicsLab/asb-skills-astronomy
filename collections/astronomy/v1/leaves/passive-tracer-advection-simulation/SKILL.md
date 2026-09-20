---
name: passive-tracer-advection-simulation
description: Use when you need to demonstrate reversibility of a low-Reynolds-number
  flow by advecting tracer fields through a time-dependent wall-driven Stokes solution
  and comparing initial vs final tracer configurations.
when_to_use_negative:
- Problem involves inertial (Navier-Stokes) effects at finite Reynolds number rather
  than Stokes (low-Re) flow
- Tracers interact chemically or have non-negligible diffusivity (advection-diffusion
  at low Pe) rather than passive advection at high Pe
- Geometry is Cartesian rather than a cylindrical annulus requiring Fourier-Chebyshev
  discretization
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_0176
tools:
- name: Dedalus
  role: Spectral PDE framework used to formulate the r-weighted Stokes problem, impose
    the arctan-regularized boundary condition via GeneralFunction, and integrate the
    coupled tracer problem with its timestepper
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Array operations for domain grids and initial tracer field definitions
- name: matplotlib
  role: Rendering snapshots of the dye tracer fields for the mixing/unmixing figure
- name: h5py
  role: Loading the dye tracer snapshots written to HDF5 analysis files for the initial-vs-final
    comparison
  repo: https://github.com/h5py/h5py
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:6c15fd4d81c03d82b51fa2d8305b927107e142bd11d390b52f5061e4d4292119
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_0176
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# passive-tracer-advection-simulation

## Summary

Simulates reversible mixing/unmixing of passive dye tracers advected by a Stokes-flow solution in a cylindrical Taylor-Couette annulus, using Dedalus spectral solves with a time-dependent inner-wall boundary condition integrated over n_rot = 8 rotations.

## When to use

Use when you need to demonstrate reversibility of a low-Reynolds-number flow by advecting tracer fields through a time-dependent wall-driven Stokes solution and comparing initial vs final tracer configurations.

## When NOT to use

- Problem involves inertial (Navier-Stokes) effects at finite Reynolds number rather than Stokes (low-Re) flow
- Tracers interact chemically or have non-negligible diffusivity (advection-diffusion at low Pe) rather than passive advection at high Pe
- Geometry is Cartesian rather than a cylindrical annulus requiring Fourier-Chebyshev discretization

## Inputs

- Cylindrical annulus domain specification (azimuthal Fourier x radial Chebyshev grids)
- Time-dependent inner-wall rotation boundary condition (arctan-regularized angular velocity profile)
- Three dye tracer initial fields (stripe patterns)
- Peclet number Pe = 1e7 and rotation count n_rot = 8

## Outputs

- Stokes velocity solution fields in the annulus
- HDF5 snapshot files of the dye tracer fields at output times
- Matplotlib figure of initial vs final tracer configurations (Fig. 14-style reversible mixing/unmixing demonstration)

## How to apply

Set up the cylindrical annulus domain in Dedalus with a Fourier basis in azimuth and a Chebyshev basis in radius. Formulate the r-weighted Stokes equations and impose the time-dependent inner-wall rotation boundary condition, regularized with an arctan profile supplied via a GeneralFunction handler. Initialize three dye tracer fields advected at Pe = 1e7 by the Stokes velocity solution, then integrate the coupled problem with the Dedalus timestepper for n_rot = 8 inner-wall rotations, writing tracer snapshots to HDF5 analysis files. Load the snapshots with h5py and compare initial versus final tracer configurations: the final field should reproduce the initial stripe pattern, demonstrating reversible unmixing.

## Related tools

- **Dedalus** (Spectral PDE framework used to formulate the r-weighted Stokes problem, impose the arctan-regularized boundary condition via GeneralFunction, and integrate the coupled tracer problem with its timestepper) — https://github.com/DedalusProject/dedalus
- **numpy** (Array operations for domain grids and initial tracer field definitions)
- **matplotlib** (Rendering snapshots of the dye tracer fields for the mixing/unmixing figure)
- **h5py** (Loading the dye tracer snapshots written to HDF5 analysis files for the initial-vs-final comparison) — https://github.com/h5py/h5py

## Examples

```
import dedalus.public as d3; solver = problem.build_solver(d3.RK222); solver.step(dT)  # integrate Stokes+tracer problem to n_rot=8
```

## Evaluation signals

- Final dye tracer field after 8 rotations reproduces the initial stripe pattern, confirming reversible unmixing
- Tracers spread into thin sheets during the counter-clockwise rotation phase, as expected for reversible mixing
- Tracer advection conserves passive-tracer structure at Pe = 1e7 (diffusion negligible relative to advection)
- Snapshot output contains all three dye tracer fields at each save time in the HDF5 analysis files

## Limitations

- Reversibility holds only in the Stokes limit; finite inertia breaks time-reversal symmetry of the flow
- High Pe (1e7) approximation of passive tracers fails if true diffusivity matters
- Time-dependent boundary conditions require special handling (arctan regularization via GeneralFunction) to remain well-posed

## Evidence

- [abstract] low-Reynolds-number-flow example: "They also illustrate its capabilities with diverse examples, including optical network dynamics, magnetized shocks in plasmas, large-scale oceanic flows, low Reynolds number flows, stellar"
- [full_text] r-weighted Stokes equations: [evidence span withheld — anchor status: not_found]
- [intro] HDF5 analysis output: "import h5py
import numpy as np"
- [readme] Dedalus spectral framework: "Dedalus is a flexible framework for solving partial differential equations using spectral methods."
