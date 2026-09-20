---
name: time-dependent-boundary-condition-regularization
description: Use when solving Stokes or Navier-Stokes flow in a cylindrical annulus
  driven by a smooth, time-dependent wall velocity (e.g. inner-wall rotation over
  n_rot rotations) in a spectral solver.
when_to_use_negative:
- Problems with steady or static boundary conditions where regularization of a time-dependent
  wall forcing is unnecessary
- Non-spectral (e.g. finite-volume) discretizations where the arctan/GeneralFunction
  spectral regularization is not applicable
- High Reynolds-number turbulent Taylor-Couette flow — this skill targets the Stokes-flow
  (low Reynolds number) reversible-mixing regime
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Dedalus
  role: Spectral PDE framework used to build the annulus problem, register the arctan-regularized
    time-dependent boundary condition via a GeneralFunction handler, and time-integrate
    the coupled Stokes-tracer problem
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Numerical array operations for defining the arctan profile and handling tracer
    field data
- name: h5py
  role: Loading the HDF5 tracer snapshots written during time integration for initial-vs-final
    comparison
  repo: https://github.com/h5py/h5py
- name: matplotlib
  role: Visualizing the dye tracer snapshots showing reversible mixing/unmixing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:4491001cdd79c2271f69ddaaaed0b8847ce2fed2b4ca6d6c9d133ef8f9ee2786
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
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# time-dependent-boundary-condition-regularization

## Summary

Imposes a time-dependent inner-wall rotation boundary condition on an r-weighted Stokes problem in a cylindrical annulus, regularizing the sharp wall forcing with an arctan profile handled through Dedalus's GeneralFunction handler. This drives the classic reversible Taylor-Couette tracer mixing/unmixing demonstration.

## When to use

Use when solving Stokes or Navier-Stokes flow in a cylindrical annulus driven by a smooth, time-dependent wall velocity (e.g. inner-wall rotation over n_rot rotations) in a spectral solver.

## When NOT to use

- Problems with steady or static boundary conditions where regularization of a time-dependent wall forcing is unnecessary
- Non-spectral (e.g. finite-volume) discretizations where the arctan/GeneralFunction spectral regularization is not applicable
- High Reynolds-number turbulent Taylor-Couette flow — this skill targets the Stokes-flow (low Reynolds number) reversible-mixing regime

## Inputs

- Cylindrical annulus domain specification (Fourier in azimuth, Chebyshev in radius)
- Time-dependent inner-wall rotation boundary condition function (regularized with an arctan profile)
- r-weighted Stokes equations formulated in Dedalus
- Initial dye tracer field configurations (three tracer fields)
- Simulation parameters: Pe=1e7, n_rot=8 inner-wall rotations

## Outputs

- Stokes velocity solution in the annulus
- HDF5 analysis files containing dye tracer snapshots over time
- Final dye tracer field after 8 rotations for comparison against the initial configuration

## How to apply

Set up a cylindrical annulus domain in Dedalus using a Fourier basis in azimuth and a Chebyshev basis in radius. Formulate the r-weighted Stokes equations and impose the time-dependent inner-wall rotation boundary condition, regularizing the discontinuous wall forcing with an arctan profile registered through a GeneralFunction handler so the spectral basis can represent it. Solve the Stokes system each step to obtain the velocity field, then advect dye tracer fields passively at Pe=1e7. Integrate the coupled problem with a Dedalus timestepper for n_rot=8 inner-wall rotations, writing tracer snapshots to HDF5 analysis files. The rationale is that the arctan regularization smooths the time-dependent boundary input into a form compatible with spectral discretization, avoiding Gibbs-type artifacts at the wall.

## Related tools

- **Dedalus** (Spectral PDE framework used to build the annulus problem, register the arctan-regularized time-dependent boundary condition via a GeneralFunction handler, and time-integrate the coupled Stokes-tracer problem) — https://github.com/DedalusProject/dedalus
- **numpy** (Numerical array operations for defining the arctan profile and handling tracer field data)
- **h5py** (Loading the HDF5 tracer snapshots written during time integration for initial-vs-final comparison) — https://github.com/h5py/h5py
- **matplotlib** (Visualizing the dye tracer snapshots showing reversible mixing/unmixing)

## Examples

```
import numpy as np; import dedalus.public as d3
```

## Evaluation signals

- The final dye tracer field after n_rot=8 rotations reproduces the initial stripe pattern, demonstrating reversible unmixing
- Snapshots show dye spreading into thin sheets during counter-clockwise rotation and then congregating back toward initial configurations
- The time-dependent inner-wall boundary condition evaluates smoothly (arctan-regularized) and is accepted by the GeneralFunction handler without spectral ringing at the wall
- Stokes velocity solution converges consistently with the Fourier-Chebyshev resolution on the annulus domain
- Tracer advection remains stable at Pe=1e7 over all 8 rotations without numerical blow-up

## Limitations

- Demonstrated only in the Stokes-flow (low Reynolds number) regime; inviscid or turbulent wall forcing may need different regularization
- The arctan profile changes the imposed wall motion slightly relative to an ideal sharp rotation schedule, so unmixing fidelity depends on the regularization width
- Tracer reversibility at Pe=1e7 relies on the Stokes velocity being solved accurately at each step; timestepper errors can degrade the return of tracers to initial positions

## Evidence

- [other] Formulate the r-weighted Stokes equations and impose the time-dependent inner-wall rotation boundary condition, regularized with an arctan profile via a GeneralFunction handler.: [evidence span withheld — anchor status: not_found]
- [other] Set up the cylindrical annulus domain in Dedalus using a Fourier-Chebyshev basis (Fourier in azimuth, Chebyshev in radius).: [evidence span withheld — anchor status: not_found]
- [intro] Dedalus is a flexible framework for solving partial differential equations using modern spectral methods.: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [readme] A flexible framework for solving PDEs with modern spectral methods.: [evidence span withheld — anchor status: external_doc]
