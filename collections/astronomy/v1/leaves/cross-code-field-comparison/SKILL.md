---
name: cross-code-field-comparison
description: Use when a benchmark problem has run through two independent solvers
  (e.g., Dedalus 2048^2 spectral modes vs. Athena finite-volume) and you must quantify
  how resolution governs accuracy against unphysical secondary instabilities.
when_to_use_negative:
- The goal is a single-code convergence study with no independent reference solver,
  so no cross-code field alignment is possible
- Only scalar or time-series diagnostics (e.g., growth rates, eigenvalues) are to
  be compared, rather than full 2D field states
- The problem's solutions are non-smooth enough that spectral methods are inappropriate,
  so the resolution-gap comparison would not be meaningful
tools:
- name: Dedalus
  role: Spectral solver used to run the 2D Kelvin-Helmholtz shear-flow simulation
    up to 2048^2 Fourier modes and write late-time snapshots via HDF5 analysis handlers
  repo: https://github.com/DedalusProject/dedalus
- name: Athena
  role: Finite-volume reference code run at 16384^2 cells to provide the late-time
    nonlinear flow state against which the spectral solution is compared
- name: h5py
  role: Reads the late-time snapshot fields saved by Dedalus's HDF5 analysis handlers
    for field-level comparison
  repo: https://github.com/h5py/h5py
- name: numpy
  role: Computes relative-difference metrics between aligned vorticity and density
    fields across the two codes
  repo: https://github.com/scipy/scipy
- name: matplotlib
  role: Renders the side-by-side resolution-comparison figure in the Fig. 9 style
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:6f4c44c9929760e07865f3d4a08720772cd689b4fa0027360c78fb2fa60a5724
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DedalusProject/dedalus
  tools:
  - Athena
  - Dedalus
  - Python
  - h5py
  - matplotlib
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cross-code-field-comparison

## Summary

Quantifies field-level agreement between a spectral-method code (Dedalus) and a finite-volume reference code (Athena) on a shared hydrodynamic benchmark, here the 2D Kelvin-Helmholtz instability, by aligning late-time snapshots at matched physical times and computing relative differences in vorticity/density fields.

## When to use

Use when a benchmark problem has run through two independent solvers (e.g., Dedalus 2048^2 spectral modes vs. Athena finite-volume) and you must quantify how resolution governs accuracy against unphysical secondary instabilities.

## When NOT to use

- The goal is a single-code convergence study with no independent reference solver, so no cross-code field alignment is possible
- Only scalar or time-series diagnostics (e.g., growth rates, eigenvalues) are to be compared, rather than full 2D field states
- The problem's solutions are non-smooth enough that spectral methods are inappropriate, so the resolution-gap comparison would not be meaningful

## Inputs

- Dedalus Kelvin-Helmholtz benchmark scripts from dedalus/src/dedalus/examples (2D shear-flow domain)
- Dedalus HDF5 snapshot files written through the analysis handlers (late-time nonlinear flow states at each resolution in the ladder)
- Finite-volume (Athena) reference fields at 16384^2 cells with matched initial conditions and timesteps
- Snapshot time stamps for aligning the two codes at the same physical time

## Outputs

- Spectral-convergence series: late-time flow states across the resolution ladder up to 2048^2 spectral modes
- Relative-difference accuracy metrics (vorticity and density fields) between the 2048^2-mode spectral case and the 16384^2-cell finite-volume case
- Side-by-side resolution-comparison figure (reproducing the Fig. 9 style comparison)
- Accuracy-metrics table

## How to apply

Retrieve the Kelvin-Helmholtz benchmark scripts from the Dedalus examples directory (dedalus/src/dedalus/examples) and verify numpy, matplotlib, and h5py import cleanly. Run the Dedalus 2D shear-flow simulation on a resolution ladder up to 2048^2 Fourier modes with matched initial conditions and timestep settings, saving late-time snapshots via the HDF5 analysis handlers. Re-run the finite-volume code at increasing resolutions up to the 16384^2-cell reference case, then align every spectral snapshot to the finite-volume snapshot at the same physical time. Compute quantitative field-level metrics, e.g. relative differences in the vorticity and density fields between the 2048^2-mode spectral state and the 16384^2-cell finite-volume state. Render a side-by-side resolution-comparison figure (as in Fig. 9 of the benchmark) and write the accuracy metrics to a table; the rationale is that spectral methods concentrate accuracy in far fewer degrees of freedom, so a spectral run matching the finite-volume reference demonstrates the resolution gap.

## Related tools

- **Dedalus** (Spectral solver used to run the 2D Kelvin-Helmholtz shear-flow simulation up to 2048^2 Fourier modes and write late-time snapshots via HDF5 analysis handlers) — https://github.com/DedalusProject/dedalus
- **Athena** (Finite-volume reference code run at 16384^2 cells to provide the late-time nonlinear flow state against which the spectral solution is compared)
- **h5py** (Reads the late-time snapshot fields saved by Dedalus's HDF5 analysis handlers for field-level comparison) — https://github.com/h5py/h5py
- **numpy** (Computes relative-difference metrics between aligned vorticity and density fields across the two codes) — https://github.com/scipy/scipy
- **matplotlib** (Renders the side-by-side resolution-comparison figure in the Fig. 9 style)

## Examples

```
import numpy as np, h5py, dedalus.public as d3; with h5py.File('kelvin_helmholtz_snapshots.h5','r') as f: vort_spectral = f['tasks/vorticity'][-1]
```

## Evaluation signals

- Late-time snapshots from both codes are aligned at the same physical time before any field comparison is computed
- The 2048^2-mode spectral late-time state is quantitatively comparable (low relative difference in vorticity/density) to the 16384^2-cell finite-volume state
- The spectral run avoids the unphysical secondary instabilities in the vortex rolls that appear in under-resolved finite-volume runs
- The resolution ladder spans coarse grids up to the finest case with matched initial conditions and timestep settings, so differences are attributable to resolution alone
- The accuracy metrics are written to a table alongside the side-by-side figure, so every claim in the figure is traceable to a number

## Limitations

- The finite-volume code needs 64x more cells per dimension (16384^2 vs. 2048^2) to reach comparable accuracy, making the reference run computationally expensive
- Under-resolved finite-volume runs exhibit unphysical secondary instabilities in the vortex rolls, which can corrupt field-level comparisons if snapshot times drift out of alignment
- Acceptance requires the spectral late-time state to be quantitatively comparable to the finite-volume state as reported in the paper's Kelvin-Helmholtz benchmark; a mismatch invalidates the reproduction
- Dedalus is designed for global spectral methods; results are easiest to transfer on single AWS instances rather than large multi-node finite-volume clusters

## Evidence

- [abstract] Dedalus spectral method efficiency rationale: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [intro] Dedalus as spectral PDE framework: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [intro] Dedalus Python interface used to build the benchmark scripts: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [methods] Dedalus runnable on a single instance for the benchmark runs: "Currently, dedalus can be run easily on single AWS instances."
