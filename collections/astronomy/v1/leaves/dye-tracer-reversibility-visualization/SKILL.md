---
name: dye-tracer-reversibility-visualization
description: Use when a low-Reynolds-number (Stokes) flow demonstration needs reversible
  tracer advection — e.g., verifying a spectral PDE solver reproduces the classic
  dye unmixing after n_rot inner-wall rotations.
when_to_use_negative:
- The problem is not a Stokes (low Reynolds number) flow — inertial or turbulent flows
  are not time-reversible and dye will not unmix
- The goal is a growth-rate or eigenvalue analysis (e.g., Rayleigh-Benard growth rates,
  Mathieu or pipe-flow eigenproblems) rather than tracer advection and visualization
- Tracer diffusivity is significant (Pe << 1e7), since molecular diffusion destroys
  the reversibility the visualization is meant to demonstrate
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Dedalus
  role: Spectral PDE framework used to solve the r-weighted Stokes equations with
    a Fourier-Chebyshev annulus basis and integrate tracer advection with its timestepper
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Array handling for tracer fields and snapshot data
- name: matplotlib
  role: Rendering the initial and final dye tracer snapshot comparisons
- name: h5py
  role: Loading dye tracer snapshots from HDF5 analysis files for the initial-vs-final
    comparison
  repo: https://github.com/h5py/h5py
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:95de2861ed6ca6757e1f1830a969abd9bd0e115d539180b082f22900c7b96fd4
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# dye-tracer-reversibility-visualization

## Summary

Reproduce and visualize reversible dye mixing/unmixing in a Taylor-Couette Stokes-flow annulus by advecting tracer fields at Pe=1e7 under a time-dependent inner-wall rotation and comparing initial vs. final tracer snapshots.

## When to use

Use when a low-Reynolds-number (Stokes) flow demonstration needs reversible tracer advection — e.g., verifying a spectral PDE solver reproduces the classic dye unmixing after n_rot inner-wall rotations.

## When NOT to use

- The problem is not a Stokes (low Reynolds number) flow — inertial or turbulent flows are not time-reversible and dye will not unmix
- The goal is a growth-rate or eigenvalue analysis (e.g., Rayleigh-Benard growth rates, Mathieu or pipe-flow eigenproblems) rather than tracer advection and visualization
- Tracer diffusivity is significant (Pe << 1e7), since molecular diffusion destroys the reversibility the visualization is meant to demonstrate

## Inputs

- Cylindrical annulus domain specification (Fourier in azimuth, Chebyshev in radius)
- Time-dependent inner-wall rotation boundary condition regularized with an arctan profile
- Three dye tracer fields with initial stripe configurations
- Peclet number Pe=1e7 and rotation count n_rot=8

## Outputs

- Dye tracer snapshot time series written to HDF5 analysis files
- Initial vs. final tracer configuration comparison (visualization showing reversible unmixing)

## How to apply

Set up a cylindrical annulus in Dedalus with a Fourier-Chebyshev basis (Fourier in azimuth, Chebyshev in radius) and formulate the r-weighted Stokes equations, imposing the time-dependent inner-wall rotation boundary condition regularized with an arctan profile via a GeneralFunction handler. Initialize three dye tracer fields advected at Pe=1e7 by the Stokes velocity solution. Integrate the coupled problem with the Dedalus timestepper for n_rot=8 inner-wall rotations, writing dye tracer snapshots to HDF5 analysis files. Reload the snapshots with h5py and render snapshots with matplotlib; the decision point is whether the final tracer field congregates back toward its initial configuration (unmixing). The rationale is that Stokes flow is time-reversible, so passive tracers mixed into thin sheets during counter-clockwise rotation must return to their initial stripe pattern.

## Related tools

- **Dedalus** (Spectral PDE framework used to solve the r-weighted Stokes equations with a Fourier-Chebyshev annulus basis and integrate tracer advection with its timestepper) — https://github.com/DedalusProject/dedalus
- **numpy** (Array handling for tracer fields and snapshot data)
- **matplotlib** (Rendering the initial and final dye tracer snapshot comparisons)
- **h5py** (Loading dye tracer snapshots from HDF5 analysis files for the initial-vs-final comparison) — https://github.com/h5py/h5py

## Examples

```
import h5py; import numpy as np; import matplotlib.pyplot as plt; import dedalus.public as d3
```

## Evaluation signals

- The final dye tracer field after 8 rotations reproduces the initial stripe pattern (unmixing confirmed)
- Intermediate snapshots show dye spreading into thin sheets during counter-clockwise rotation before re-congregating
- HDF5 analysis files contain dye tracer snapshots at each write interval for all three tracer fields
- The inner-wall rotation boundary condition applies the intended arctan-regularized time-dependent profile
- Tracer advection runs at the specified Pe=1e7 with no visible numerical diffusion smearing the sheets

## Limitations

- Reversibility depends on the Stokes (low Reynolds number) regime; at nonzero inertia the unmixing degrades
- Verification is qualitative — it relies on visual comparison of initial versus final tracer configurations in the rendered snapshots
- The unmixing is only demonstrated after a fixed rotation count (n_rot=8); other rotation counts were not validated in this task

## Evidence

- [abstract] Dedalus translates plain-text equations into efficient and parallelized solvers: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [abstract] low Reynolds number flows among Dedalus examples: "low Reynolds number flows, stellar and atmospheric waves, and diamagnetic levitation."
- [intro] Dedalus is a flexible framework: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [intro] Dedalus is written primarily in Python: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [intro] h5py import for snapshot loading: "import h5py
import numpy as np"
- [readme] Feedstock summary of Dedalus scope: [evidence span withheld — anchor status: external_doc]
