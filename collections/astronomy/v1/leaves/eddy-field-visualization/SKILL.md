---
name: eddy-field-visualization
description: Use when HDF5 snapshot output from a spectrally-discretized geophysical-flow
  simulation (e.g. Dedalus quasigeostrophic run with 256x128x32 modes) must be inspected
  qualitatively for coherent eddy structures in its saturated state.
when_to_use_negative:
- Input is a scalar time series or statistics table with no 2D/3D spatial fields to
  slice
- Fields are still in spectral coefficient space and no grid-space slice arrays are
  available in the HDF5 output
- The goal is quantitative eddy detection or spectral diagnostics rather than qualitative
  visual inspection of saturated-state structure
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: Dedalus
  role: Produces the quasigeostrophic simulation snapshots via symbolic equation entry,
    Ekman pumping boundary conditions, and MPI-parallel sparse solvers run to t=200
  repo: https://github.com/DedalusProject/dedalus
- name: h5py
  role: Python wrapper used to open the simulation's HDF5 snapshot files and extract
    the p and w task fields at final time
  repo: https://github.com/h5py/h5py
- name: numpy
  role: Array manipulation of the extracted p and w fields prior to plotting
- name: matplotlib
  role: Renders the 2D slice plots that expose the compact eddy structures of the
    saturated state
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:c61bb3f27b57ad0791d209a5546e764bd32f0891936c9c863692a752d8cc67c2
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
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# eddy-field-visualization

## Summary

Render 2D slice plots of potential vorticity (p) and buoyancy perturbation (w) fields from HDF5 snapshots of a 3D quasigeostrophic Dedalus simulation, revealing the compact-eddy structure of the statistically saturated state at t=200.

## When to use

Use when HDF5 snapshot output from a spectrally-discretized geophysical-flow simulation (e.g. Dedalus quasigeostrophic run with 256x128x32 modes) must be inspected qualitatively for coherent eddy structures in its saturated state.

## When NOT to use

- Input is a scalar time series or statistics table with no 2D/3D spatial fields to slice
- Fields are still in spectral coefficient space and no grid-space slice arrays are available in the HDF5 output
- The goal is quantitative eddy detection or spectral diagnostics rather than qualitative visual inspection of saturated-state structure

## Inputs

- Dedalus HDF5 snapshot files (.h5) containing the w (buoyancy) and p (potential vorticity) task fields
- Simulation metadata: 256x128x32 mode spectrally-discretized domain, final integration time t=200

## Outputs

- 2D matplotlib slice plots of final-time potential vorticity (p)
- 2D matplotlib slice plots of final-time buoyancy perturbation (w) showing compact eddy structures

## How to apply

Load the simulation's HDF5 output files with h5py and extract the final-time (t=200) potential vorticity (p) and buoyancy perturbation (w) fields stored on the spectrally-discretized 256x128x32 domain. Transform or index the stored slices so horizontal structures are visible, then render them as matplotlib slice (imshow/contour) plots. Inspect the rendered fields for a 'sea of compact eddies' that form, merge, and break up over several dynamical times — the reported signature of the statistically saturated state in Fig. 13. Validate qualitatively by comparing the rendered eddy patterns against the reported example behavior, since no quantitative threshold is defined for eddy identification.

## Related tools

- **Dedalus** (Produces the quasigeostrophic simulation snapshots via symbolic equation entry, Ekman pumping boundary conditions, and MPI-parallel sparse solvers run to t=200) — https://github.com/DedalusProject/dedalus
- **h5py** (Python wrapper used to open the simulation's HDF5 snapshot files and extract the p and w task fields at final time) — https://github.com/h5py/h5py
- **numpy** (Array manipulation of the extracted p and w fields prior to plotting)
- **matplotlib** (Renders the 2D slice plots that expose the compact eddy structures of the saturated state)

## Examples

```
import h5py, numpy as np, matplotlib.pyplot as plt; f=h5py.File('slices/slices.h5','r'); plt.imshow(np.real(f['tasks']['p'][0])); plt.savefig('pv_slice_t200.png')
```

## Evaluation signals

- Time integration ran to t=200 without failure, so snapshots correspond to the statistically saturated state
- HDF5 snapshot files exist and contain both the w and p task fields
- Rendered slices show the compact eddies characteristic of the saturated state (qualitative expert check against the reported Fig. 13 behavior)
- Field shapes match the 256x128x32 mode domain discretization before plotting

## Limitations

- Validation is qualitative (expert visual check against the reported example behavior); there is no quantitative threshold for eddy identification
- Requires the full MPI-parallel time integration to have completed to t=200; incomplete runs will not show saturated-state eddies
- Slice plots only expose 2D sections of the 3D field, so along-depth eddy structure is not directly visible

## Evidence

- [results] compact eddies finding: [evidence span withheld — anchor status: not_found]
- [intro] dedalus framework quote: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [readme] h5py quote: "`h5py` is a thin, pythonic wrapper around `HDF5 <https://www.hdfgroup.org/solutions/hdf5/>`_,
which runs on Python 3"
- [intro] matplotlib import quote: "import numpy as np
import matplotlib.pyplot as plt
import dedalus.public as d3"
