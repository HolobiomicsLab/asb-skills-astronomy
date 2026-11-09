---
name: resolution-convergence-analysis
description: Use when comparing spectral vs. finite-volume accuracy for an instability
  benchmark, or when determining the minimum grid/mode count (e.g., 2048^2 spectral
  modes vs. 16384^2 finite-volume cells) needed to suppress unphysical secondary instabilities.
when_to_use_negative:
- When the quantity of interest is a linear eigenvalue or growth rate that converges
  directly via sparse/dense solves (e.g., Rayleigh-Benard growth rates or pipe-flow
  eigenvalues) rather than a nonlinear late-time state requiring resolution comparison
- When no finite-volume or higher-resolution reference solution exists to anchor the
  accuracy comparison
- When the problem is not resolution-limited — e.g., if error is dominated by timestep
  or boundary conditions rather than spatial discretization
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Dedalus
  role: Runs the spectral-method simulations across the resolution ladder and writes
    HDF5 snapshots via analysis handlers
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Computes relative-difference accuracy metrics between spectral and finite-volume
    fields
- name: h5py
  role: Reads the HDF5 analysis-handler snapshots for snapshot-time alignment and
    field comparison
  repo: https://github.com/h5py/h5py
- name: matplotlib
  role: Renders the side-by-side resolution-comparison figure
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:7b0e32d64e80dac48b2e0012b06b71a7e63b6d25d9e8bd75d4da8f66a62467ca
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# resolution-convergence-analysis

## Summary

Builds a spectral-convergence series by running the same PDE simulation (e.g., the 2D Kelvin-Helmholtz shear-flow benchmark in Dedalus) at a ladder of resolutions and comparing late-time nonlinear states against a higher-resolution or finite-volume reference to quantify the accuracy gained per degree of freedom.

## When to use

Use when comparing spectral vs. finite-volume accuracy for an instability benchmark, or when determining the minimum grid/mode count (e.g., 2048^2 spectral modes vs. 16384^2 finite-volume cells) needed to suppress unphysical secondary instabilities.

## When NOT to use

- When the quantity of interest is a linear eigenvalue or growth rate that converges directly via sparse/dense solves (e.g., Rayleigh-Benard growth rates or pipe-flow eigenvalues) rather than a nonlinear late-time state requiring resolution comparison
- When no finite-volume or higher-resolution reference solution exists to anchor the accuracy comparison
- When the problem is not resolution-limited — e.g., if error is dominated by timestep or boundary conditions rather than spatial discretization

## Inputs

- Kelvin-Helmholtz benchmark scripts from dedalus/src/dedalus/examples (2D shear-flow domain)
- Resolution ladder configuration (coarse grids up to 2048^2 spectral modes) with matched initial conditions and timesteps
- Finite-volume reference solution (Athena, 16384^2 cells) snapshots at matched physical times

## Outputs

- Late-time HDF5 snapshots for each resolution in the spectral-convergence series
- Field-level agreement metrics table (relative differences in vorticity/density vs. reference)
- Side-by-side resolution-comparison figure (Fig. 9 style)

## How to apply

Retrieve the Kelvin-Helmholtz benchmark scripts from the Dedalus examples directory and verify numpy, matplotlib, and h5py are importable before launching runs. Execute the 2D shear-flow spectral simulation at a ladder of increasing resolutions — coarse grids up to the 2048^2-mode case — keeping initial conditions and timestep settings matched across the series, and save late-time snapshots through HDF5 analysis handlers. Compute a finite-volume reference (e.g., Athena at 16384^2 cells) and align snapshots at the same physical time. Quantify field-level agreement with relative differences in vorticity/density fields between the 2048^2-mode spectral case and the finite-volume reference, then render a side-by-side resolution-comparison figure and write accuracy metrics to a table. Accept the reproduction if the 2048^2-mode spectral late-time state is quantitatively comparable to the 16384^2-cell finite-volume state, matching the paper's Kelvin-Helmholtz benchmark.

## Related tools

- **Dedalus** (Runs the spectral-method simulations across the resolution ladder and writes HDF5 snapshots via analysis handlers) — https://github.com/DedalusProject/dedalus
- **numpy** (Computes relative-difference accuracy metrics between spectral and finite-volume fields)
- **h5py** (Reads the HDF5 analysis-handler snapshots for snapshot-time alignment and field comparison) — https://github.com/h5py/h5py
- **matplotlib** (Renders the side-by-side resolution-comparison figure)

## Examples

```
python3 -m pip install dedalus h5py matplotlib numpy && python3 kh_benchmark.py --resolution 2048
```

## Evaluation signals

- 2048^2-mode spectral late-time state is quantitatively comparable to the 16384^2-cell finite-volume state (relative differences in vorticity/density fields small)
- Spectral cases at lower resolutions in the ladder reproduce the unphysical secondary instabilities in the vortex rolls that vanish at 2048^2 modes
- Snapshots across cases are aligned at the same physical time before differencing
- Accuracy metrics table is populated for every rung of the resolution ladder with a consistent metric definition

## Limitations

- Matching Dedalus's 2048^2-mode accuracy required 16384^2 finite-volume cells — a factor of 64 more degrees of freedom — so finite-volume cross-checks at high resolution are computationally expensive
- Spectral methods can produce spurious secondary instabilities in the vortex rolls at insufficient resolution, which could be mistaken for physics if only coarse grids are run
- Results are benchmark-specific (2D Kelvin-Helmholtz); convergence rates do not transfer to other equations or dimensionalities without rerunning the ladder

## Evidence

- [other] Spectral accuracy vs. finite volume: [evidence span withheld — anchor status: not_found]
- [intro] Dedalus spectral methods scope: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [intro] Snapshot output mechanism: "import h5py
import numpy as np"
- [readme] Library stack: "HDF5 for Python
===============
`h5py` is a thin, pythonic wrapper around `HDF5"
- [abstract] Dedalus capability claim: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
