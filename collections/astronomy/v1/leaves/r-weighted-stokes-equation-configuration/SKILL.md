---
name: r-weighted-stokes-equation-configuration
description: Use when reproducing low-Reynolds-number (Stokes) flow in a cylindrical
  annulus with a rotating inner wall, or verifying reversible tracer mixing after
  a fixed number of wall rotations.
when_to_use_negative:
- The problem involves inertia (finite Reynolds number) rather than the Stokes limit
  — use the full Navier-Stokes formulation instead.
- The geometry is Cartesian or spherical rather than a cylindrical annulus requiring
  the r-weighted (cylindrical) formulation.
- The boundary condition is steady wall rotation, so the arctan regularization of
  a time-dependent start via GeneralFunction is unnecessary.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0153
tools:
- name: Dedalus
  role: Framework for symbolically specifying the r-weighted Stokes equations on the
    Fourier-Chebyshev annulus domain and integrating them with its timestepper
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Array operations for grid setup and tracer field initialization
- name: matplotlib
  role: Rendering dye tracer snapshots for comparison of initial versus final configurations
- name: h5py
  role: Loading tracer snapshot fields from Dedalus HDF5 analysis files for validation
  repo: https://github.com/h5py/h5py
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:2e687f672113211aa11a82156d47f8078d97da1425ab03d8ef71ceda3237e6cb
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# r-weighted Stokes equation configuration

## Summary

Configure the r-weighted Stokes equations in Dedalus for a cylindrical Taylor-Couette annulus with a time-dependent inner-wall rotation, and advect dye tracers to demonstrate reversible mixing/unmixing as in Fig. 14.

## When to use

Use when reproducing low-Reynolds-number (Stokes) flow in a cylindrical annulus with a rotating inner wall, or verifying reversible tracer mixing after a fixed number of wall rotations.

## When NOT to use

- The problem involves inertia (finite Reynolds number) rather than the Stokes limit — use the full Navier-Stokes formulation instead.
- The geometry is Cartesian or spherical rather than a cylindrical annulus requiring the r-weighted (cylindrical) formulation.
- The boundary condition is steady wall rotation, so the arctan regularization of a time-dependent start via GeneralFunction is unnecessary.

## Inputs

- Cylindrical annulus domain specification (radii of inner/outer walls, azimuthal/radial grid resolutions)
- Time-dependent inner-wall angular velocity boundary condition (arctan-regularized profile)
- Tracer advection parameters (Peclet number Pe = 1e7, number of wall rotations n_rot = 8)

## Outputs

- Stokes velocity solution fields in the annulus
- Dye tracer snapshots written to HDF5 analysis files at intervals over 8 rotations
- Initial-versus-final tracer comparison plots showing reversible unmixing (Fig. 14-style)

## How to apply

Set up the cylindrical annulus domain with a Fourier basis in azimuth and a Chebyshev basis in radius, then formulate the r-weighted Stokes equations in Dedalus. Impose the time-dependent inner-wall rotation boundary condition, regularizing the impulsive start with an arctan profile supplied through a GeneralFunction handler. Initialize three dye tracer fields advected passively at Peclet number 1e7 by the Stokes velocity solution, and integrate the coupled problem with the Dedalus timestepper for n_rot = 8 inner-wall rotations, writing tracer snapshots to HDF5 analysis files. Load the snapshots with h5py and compare final versus initial tracer configurations; the reversible Stokes flow should spread dye into thin sheets during counter-clockwise rotation and then return it to its initial stripe pattern, demonstrating unmixing.

## Related tools

- **Dedalus** (Framework for symbolically specifying the r-weighted Stokes equations on the Fourier-Chebyshev annulus domain and integrating them with its timestepper) — https://github.com/DedalusProject/dedalus
- **numpy** (Array operations for grid setup and tracer field initialization)
- **matplotlib** (Rendering dye tracer snapshots for comparison of initial versus final configurations)
- **h5py** (Loading tracer snapshot fields from Dedalus HDF5 analysis files for validation) — https://github.com/h5py/h5py

## Examples

```
import dedalus.public as d3; solver = problem.build_solver(d3.RK222); solver.solve(time_sim.stop); tracer_file = d3.HDF5File('tracer_snapshots.h5')
```

## Evaluation signals

- After n_rot = 8 inner-wall rotations, the final dye tracer field reproduces the initial stripe pattern (reversible unmixing).
- Mid-rotation snapshots show dye spread into thin sheets during counter-clockwise rotation, confirming active stirring before unmixing.
- The Stokes velocity solution satisfies no-slip on both walls, with the inner-wall speed matching the arctan-regularized time-dependent profile.
- HDF5 analysis files contain tracer snapshots at the expected write cadence and can be opened with h5py without errors.

## Limitations

- Reversibility holds only in the Stokes limit; the tracer advection Peclet number (1e7) must be high enough that diffusion does not destroy the recovered stripe pattern.
- The time-dependent inner-wall boundary condition must be regularized (arctan profile via GeneralFunction); an abrupt impulsive start may cause numerical problems.
- Eight wall rotations is the validated integration length; other rotation counts are not benchmarked in this example.

## Evidence

- [results] Stokes-flow Taylor-Couette annulus driven by time-dependent inner-wall boundary condition over n_rot = 8 rotations: [evidence span withheld — anchor status: not_found]
- [results] dye unmixing back to initial configuration: [evidence span withheld — anchor status: not_found]
- [results] tracers advected at Pe=1e7: [evidence span withheld — anchor status: not_found]
- [intro] Dedalus spectral framework: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [intro] HDF5 snapshot loading: "import h5py
import numpy as np"
