---
name: simulation-field-visualization
description: Use when a Dedalus (or similar spectral-solver) simulation has written
  HDF5 snapshot files and a 2D physical field such as density must be extracted and
  rendered as an image to confirm resolved flow features.
when_to_use_negative:
- Input is raw experimental or tabular data rather than HDF5 snapshots produced by
  a spectral PDE solver like Dedalus
- The goal is quantitative eigenvalue or growth-rate validation (e.g. Rayleigh-Benard
  or Mathieu problems) rather than qualitative field rendering
- The field of interest is 1D or requires volumetric 3D rendering; this skill covers
  2D matplotlib field maps
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: h5py
  role: Opens the Dedalus-written HDF5 snapshot files and extracts the density field
    at t=0.5
  repo: https://github.com/h5py/h5py
- name: matplotlib
  role: Renders the extracted 2D density field as a color map for visual shock-feature
    inspection
- name: Dedalus
  role: Produces the Orszag-Tang vortex MHD simulation snapshots (RK443 timestepper,
    CFL-restricted adaptive timestep, MPI-parallelized) that are visualized
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Array handling for the extracted field data during loading and plotting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:c9d754b0bf35afadbe1b6eeafa0145d6c872529c92fbeb6357cfd689b3bf9991
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/h5py/h5py
  tools:
  - Dedalus
  - MPI
  - Python
  - h5py
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# simulation-field-visualization

## Summary

Render 2D field variables (e.g. density in an Orszag-Tang vortex MHD simulation) from Dedalus HDF5 snapshot files as matplotlib maps, to visually verify that sharp shock profiles and shock-shock interaction features are resolved at a target simulation time.

## When to use

Use when a Dedalus (or similar spectral-solver) simulation has written HDF5 snapshot files and a 2D physical field such as density must be extracted and rendered as an image to confirm resolved flow features.

## When NOT to use

- Input is raw experimental or tabular data rather than HDF5 snapshots produced by a spectral PDE solver like Dedalus
- The goal is quantitative eigenvalue or growth-rate validation (e.g. Rayleigh-Benard or Mathieu problems) rather than qualitative field rendering
- The field of interest is 1D or requires volumetric 3D rendering; this skill covers 2D matplotlib field maps

## Inputs

- Dedalus HDF5 snapshot files (.h5) from a compressible MHD Orszag-Tang vortex run at t=0.5
- Density field array (4096x4096 grid) stored in the snapshots
- Simulation parameters (timestepper, CFL-restricted adaptive timestep, dealiasing setting) for provenance

## Outputs

- 2D density field map rendered with matplotlib.pyplot showing shock profiles
- Visual confirmation of shock-shock interaction features in the upper-left and lower-right domain corners

## How to apply

First, load the written HDF5 snapshot files with h5py and select the snapshot at the target simulation time (here t=0.5 of the Orszag-Tang vortex run, computed on a 4096x4096 periodic 2D Fourier domain with 3/2 dealiasing). Extract the density field array from the snapshot. Render the field with matplotlib.pyplot as a 2D map (imshow/pcolormesh), inspecting for very sharp density shocks and the shock-shock interaction features expected in the upper-left and lower-right corners of the domain. Because Dedalus solvers are automatically parallelized using MPI, snapshots may be sharded across MPI ranks, so verify that all writes are merged before plotting; if reading files produced by parallel runs, ensure h5py was built with parallel HDF5 support (configure with --enable-parallel against mpiicc/mpiifort). Correct application is judged by the presence of the corner interaction features and non-oscillatory, resolved shock profiles.

## Related tools

- **h5py** (Opens the Dedalus-written HDF5 snapshot files and extracts the density field at t=0.5) — https://github.com/h5py/h5py
- **matplotlib** (Renders the extracted 2D density field as a color map for visual shock-feature inspection)
- **Dedalus** (Produces the Orszag-Tang vortex MHD simulation snapshots (RK443 timestepper, CFL-restricted adaptive timestep, MPI-parallelized) that are visualized) — https://github.com/DedalusProject/dedalus
- **numpy** (Array handling for the extracted field data during loading and plotting)

## Examples

```
import h5py; import matplotlib.pyplot as plt; f = h5py.File('snapshots.h5'); plt.imshow(f['tasks/rho'][-1]); plt.show()
```

## Evaluation signals

- The rendered density map at t=0.5 shows very sharp shock profiles without grid-scale oscillations
- Shock-shock interaction features appear in the upper-left and lower-right corners of the domain, matching the expected Orszag-Tang phenomenology
- The extracted array has the expected 4096x4096 shape on the periodic 2D Fourier domain
- HDF5 files open cleanly with h5py; for MPI-parallel runs, all rank writes are present (no missing shards)
- Nonlinear terms were dealiased at 3/2, so no aliasing-driven artifacts contaminate the plotted field

## Limitations

- Resolving high-Mach-number shocks spectrally requires appropriate regularization (explicit diffusion terms for momentum, heat, and magnetic field equations) and very high resolution (4096x4096 modes), making the run computationally expensive and MPI-parallelization effectively mandatory
- Dedalus solvers are automatically parallelized using MPI, so snapshot files may be distributed; h5py must be built against parallel HDF5 (CC=mpiicc, --enable-parallel) to read such outputs
- Visualization is qualitative: it confirms feature presence but does not by itself quantify shock-capturing accuracy

## Evidence

- [abstract] Magnetized shocks in plasmas illustrated as a Dedalus capability: "They also illustrate its capabilities with diverse examples, including optical network dynamics, magnetized shocks in plasmas, large-scale oceanic flows, low Reynolds number flows, stellar and"
- [intro] Dedalus is Python-based with an easy interface: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [methods] Parallel HDF5 build needed for MPI-parallel snapshot IO: "./configure --prefix=$BUILD_HOME CC=mpiicc CXX=mpiicpc F77=mpiifort \
                         --enable-shared --enable-parallel"
- [readme] Feedstock summary confirming Dedalus's spectral PDE framework: [evidence span withheld — anchor status: external_doc]
