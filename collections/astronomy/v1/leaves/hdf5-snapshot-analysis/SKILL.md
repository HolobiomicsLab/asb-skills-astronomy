---
name: hdf5-snapshot-analysis
description: Use when a Dedalus simulation run has produced HDF5 snapshot files through
  its analysis/simulation output handlers and the goal is to extract named task fields
  (density, vorticity, buoyancy w, potential vorticity p) and compare or render them
  at specific simulation times.
when_to_use_negative:
- The raw simulation output has not been generated yet — analysis is a post-processing
  step after the Dedalus time integration (SBDF2 or RK443) has written snapshots
- The data are scalar time series or eigenvalue tables (e.g. Rayleigh-Benard growth
  rates at Rayleigh=1710, Prandtl=1) rather than spatial field snapshots
- The HDF5 files were written by a non-Dedalus pipeline with a different internal
  layout than Dedalus's analysis-handler tasks structure
edam_operation: http://edamontology.org/operation_2409
tools:
- name: h5py
  role: Opens the HDF5 snapshot files written by Dedalus's simulation output handler
    and extracts task datasets (density, w, p, dye tracers) into numpy arrays
  repo: https://github.com/h5py/h5py
- name: Dedalus
  role: Produces the HDF5 snapshot output during time integration of the spectral
    PDE solvers; its example scripts are located in dedalus/src/dedalus/examples
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Array operations on extracted fields, e.g. computing relative differences
    against reference finite-volume states
- name: matplotlib
  role: Renders extracted 2D fields as slice maps to visually confirm reported features
    (shocks, eddies, tracer unmixing)
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:f6f174ed9eee16d0a0a704b0261dff1cc8a795c2f4c433838f176519fa0d5617
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hdf5-snapshot-analysis

## Summary

Post-processes field snapshots written by Dedalus's HDF5 analysis handlers (via h5py) into quantitative field extractions and matplotlib renderings, e.g. the Orszag-Tang density field at t=0.5 or quasigeostrophic w/p slices at t=200.

## When to use

Use when a Dedalus simulation run has produced HDF5 snapshot files through its analysis/simulation output handlers and the goal is to extract named task fields (density, vorticity, buoyancy w, potential vorticity p) and compare or render them at specific simulation times.

## When NOT to use

- The raw simulation output has not been generated yet — analysis is a post-processing step after the Dedalus time integration (SBDF2 or RK443) has written snapshots
- The data are scalar time series or eigenvalue tables (e.g. Rayleigh-Benard growth rates at Rayleigh=1710, Prandtl=1) rather than spatial field snapshots
- The HDF5 files were written by a non-Dedalus pipeline with a different internal layout than Dedalus's analysis-handler tasks structure

## Inputs

- HDF5 snapshot files written by Dedalus analysis handlers / simulation output handler (e.g. tasks containing density, vorticity, w, p, dye tracer fields)
- Target simulation times for snapshot alignment (e.g. t=0.5, t=200)
- Reference fields for comparison (e.g. finite-volume Athena 16384^2-cell density/vorticity)

## Outputs

- numpy arrays of extracted task fields at specified times
- 2D matplotlib slice/render plots of density, vorticity, or potential vorticity fields
- Quantitative field-level agreement metrics (e.g. relative differences in vorticity/density fields) and resolution-comparison tables

## How to apply

After a Dedalus run finishes (e.g. the 4096x4096-mode Orszag-Tang MHD case integrated with RK443 and CFL-restricted timestepping to t=0.5, or the 256x128x32-mode quasigeostrophic case to t=200), open the written HDF5 files with h5py and pull task datasets (e.g. density at t=0.5) into numpy arrays. Select snapshots aligned at the same physical time when comparing resolutions, as in the Kelvin-Helmholtz accuracy comparison against the 16384^2-cell Athena finite-volume reference. Render extracted 2D fields with matplotlib.pyplot as slice maps, checking for paper-reported features such as sharp density shocks with shock-shock interaction in the upper-left and lower-right corners, compact eddies in the saturated state, or dye tracers unmixing back to their initial stripe pattern after n_rot=8 rotations. For quantitative checks, compute field-level agreement metrics such as relative differences in vorticity/density fields between spectral and finite-volume cases.

## Related tools

- **h5py** (Opens the HDF5 snapshot files written by Dedalus's simulation output handler and extracts task datasets (density, w, p, dye tracers) into numpy arrays) — https://github.com/h5py/h5py
- **Dedalus** (Produces the HDF5 snapshot output during time integration of the spectral PDE solvers; its example scripts are located in dedalus/src/dedalus/examples) — https://github.com/DedalusProject/dedalus
- **numpy** (Array operations on extracted fields, e.g. computing relative differences against reference finite-volume states)
- **matplotlib** (Renders extracted 2D fields as slice maps to visually confirm reported features (shocks, eddies, tracer unmixing))

## Examples

```
import h5py, numpy as np; f = h5py.File('snapshots.h5', 'r'); density = np.array(f['tasks']['density'][0])
```

## Evaluation signals

- The HDF5 files exist and contain the expected task fields (e.g. w and p for the quasigeostrophic case; density for Orszag-Tang) at the target time
- The rendered density map at t=0.5 shows the reported sharp shock profiles and shock-shock interaction features in the upper-left and lower-right corners
- The quasigeostrophic fields at t=200 show the compact eddies of the statistically saturated state (qualitative expert check)
- For the Kelvin-Helmholtz comparison, the 2048^2-mode spectral late-time state is quantitatively comparable (relative field differences) to the 16384^2-cell finite-volume reference
- For the Taylor-Couette case, the final dye tracer field after n_rot=8 rotations reproduces the initial stripe pattern

## Limitations

- For MPI-parallel runs, h5py/HDF5 must be built with parallel support (./configure --enable-parallel) or snapshot I/O will fail on distributed runs
- Very high-resolution cases (4096x4096 modes with 3/2 dealiasing, 16384^2-cell references) produce large snapshot files, so slice selection at matched physical times is needed before metric computation
- Validation of some examples (e.g. the saturated quasigeostrophic eddies) is qualitative and relies on expert visual comparison rather than numeric thresholds

## Evidence

- [abstract] Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods.: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [intro] import h5py
import numpy as np: "import h5py
import numpy as np"
- [readme] h5py is a thin, pythonic wrapper around HDF5: [evidence span withheld — anchor status: external_doc]
- [readme] Source code: https://github.com/h5py/h5py: [evidence span withheld — anchor status: external_doc]
- [methods] Code repository: https://github.com/DedalusProject/dedalus: "* Code repository: https://github.com/DedalusProject/dedalus"
