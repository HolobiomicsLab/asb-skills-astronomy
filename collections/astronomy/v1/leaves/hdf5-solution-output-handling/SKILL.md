---
name: hdf5-solution-output-handling
description: Use when a multi-stage Dedalus workflow must pass intermediate solver
  state (e.g., a solved static atmosphere background) from one stage (NLBVP Newton
  solve) to another (eigenvalue solves) via files.
when_to_use_negative:
- Input is already a loaded Dedalus field object in memory with no need for cross-process
  persistence
- Output must be a plain-text or tabular intermediate (e.g., a CSV of eigenfrequencies);
  HDF5 is for structured solver/background state
- The workflow is single-stage and the eigenproblem can be built directly from the
  NLBVP solution without a file round-trip
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3369
tools:
- name: h5py
  role: Python wrapper used to write/read the HDF5 file holding the solved background
    fields
  repo: https://github.com/h5py/h5py
- name: Dedalus
  role: Produces the NLBVP solution fields and consumes the HDF5-backed state for
    the perturbation eigenproblem
  repo: https://github.com/DedalusProject/dedalus
- name: HDF5
  role: Underlying parallel-capable file format; the paper documents building HDF5
    with MPI support for Dedalus I/O
  repo: https://github.com/h5py/h5py
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:506039985fa1c9a86af47416b891b47fba45cfafdd838628ec10d06cd87d9961
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/h5py/h5py
  tools:
  - Dedalus
  - HDF5
  - Python
  - h5py
  - matplotlib
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3369
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hdf5-solution-output-handling

## Summary

Persist and reload Dedalus simulation state (e.g., radiative-atmosphere NLBVP background fields) in HDF5 format using h5py, mirroring Dedalus's own file-based output conventions.

## When to use

Use when a multi-stage Dedalus workflow must pass intermediate solver state (e.g., a solved static atmosphere background) from one stage (NLBVP Newton solve) to another (eigenvalue solves) via files.

## When NOT to use

- Input is already a loaded Dedalus field object in memory with no need for cross-process persistence
- Output must be a plain-text or tabular intermediate (e.g., a CSV of eigenfrequencies); HDF5 is for structured solver/background state
- The workflow is single-stage and the eigenproblem can be built directly from the NLBVP solution without a file round-trip

## Inputs

- Converged NLBVP solution fields (static radiative-atmosphere background: e.g., mean temperature/pressure/density profiles on the Legendre spectral grid)
- Dedalus solver state objects (distributor, basis, field handles) from the Newton iteration stage

## Outputs

- HDF5 file (.h5) containing named background-field datasets readable by h5py and by Dedalus's file handling
- Re-loaded background field objects used to construct the linear perturbation eigenproblem for 20 horizontal wavenumbers

## How to apply

After solving the plain-text NLBVP for the static radiative atmosphere (Eddington radiative closure with Kramer-like opacity) via Newton iteration, serialize the converged background fields to an HDF5 file with h5py so the downstream linear perturbation eigenproblem can load them instead of re-solving. HDF5 is the I/O backbone Dedalus itself uses for simulation output, so writing background fields this way keeps artifacts interchangeable with Dedalus's own readers and checkpoint conventions. When loading, open the file read-only with h5py and feed the field arrays into the perturbation problem's basis/setup before calling solver.solve_sparse for each of the 20 horizontal wavenumbers. Keep the file grouped by field name and verify numeric types/datasets survive a round-trip before launching the eigenvalue stage.

## Related tools

- **h5py** (Python wrapper used to write/read the HDF5 file holding the solved background fields) — https://github.com/h5py/h5py
- **Dedalus** (Produces the NLBVP solution fields and consumes the HDF5-backed state for the perturbation eigenproblem) — https://github.com/DedalusProject/dedalus
- **HDF5** (Underlying parallel-capable file format; the paper documents building HDF5 with MPI support for Dedalus I/O) — https://github.com/h5py/h5py

## Examples

```
import h5py; f = h5py.File('atmosphere_background.h5', 'w'); f.create_dataset('T', data=T0); f.close()
```

## Evaluation signals

- Reopening the HDF5 file with h5py returns datasets whose shapes match the spectral grid (N_green_FUNC points along the Legendre basis) and whose values equal the in-memory fields element-wise
- The eigenproblem built from the re-loaded background reproduces the same eigenvalues as one built directly from the in-memory NLBVP solution (identical spectral spacings)
- File is readable by Dedalus's own file-handling utilities (writer/reader conventions), not just raw h5py
- Downstream solve_sparse runs for all 20 horizontal wavenumbers without KeyError/shape errors attributable to missing background datasets

## Limitations

- For parallel runs, plain h5py writes must align with an MPI-enabled HDF5 build (the paper builds HDF5 with --enable-parallel); serial h5py can conflict with parallel HDF5 installations
- Dedalus's HDF5 output conventions (group layout, metadata) must be respected or Dedalus's own readers will not recognize the file

## Evidence

- [intro] h5py write of background fields: "import h5py
import numpy as np"
- [methods] HDF5 parallel build for Dedalus I/O: "./configure --prefix=$BUILD_HOME CC=mpiicc CXX=mpiicpc F77=mpiifort \
                         --enable-shared --enable-parallel"
- [methods] h5py repository: "git clone git://github.com/h5py/h5py.git"
- [readme] HDF5 for Python description: [evidence span withheld — anchor status: external_doc]
- [abstract] Dedalus solves and translates plain-text equations into spectral solvers: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
