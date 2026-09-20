---
name: eigenspectrum-spacing-and-mode-trapping-visualization
description: Use when a computed wave eigenproblem must be checked against theoretical
  spectral regularities — constant frequency spacing of acoustic modes, constant period
  spacing of gravity modes — or when mode trapping regions must be visualized as eigenfrequency
  vs. horizontal wavenumber.
when_to_use_negative:
- The problem is a time-dependent initial-value simulation rather than a targeted
  sparse eigenvalue problem — use Dedalus's IVP solvers instead of solve_sparse.
- The eigenmodes cannot be organized into acoustic/gravity branches with identifiable
  frequencies near theoretical targets, since the spacing diagnostic requires mode
  identification per branch.
- Validation requires dense eigenvalue sweeps over all subproblems (e.g. small test
  problems like the Mathieu equation) rather than targeted sparse solves near a mode
  frequency.
edam_operation: http://edamontology.org/operation_0337
tools:
- name: Dedalus
  role: Formulates the plain-text NLBVP and linear eigenproblem and performs the targeted
    sparse eigenvalue solves (solver.solve_sparse) per horizontal wavenumber
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Extracts eigenfrequencies and computes the acoustic frequency spacings and
    gravity period spacings across the 20 wavenumbers
- name: matplotlib
  role: Plots the eigenspectrum (eigenfrequency vs. horizontal wavenumber), marking
    the acoustic and gravity branches
- name: h5py
  role: Writes the background atmosphere fields from the Newton-iterated NLBVP solution
    to an HDF5 file for reuse in the eigenproblem
  repo: https://github.com/h5py/h5py
- name: scipy
  role: Supporting numerical routines for the eigenproblem setup in the Dedalus Python
    environment
  repo: https://github.com/scipy/scipy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:fc4395e4b7e3966e2dc96180d3a9551e9480468db33b1caa90744e13f3258f77
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
  - scipy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# eigenspectrum-spacing-and-mode-trapping-visualization

## Summary

Computes linear wave eigenmodes of a stratified radiative atmosphere with Dedalus's sparse eigenvalue solver over 20 horizontal wavenumbers and visualizes the eigenspectrum to verify acoustic-mode frequency spacing, gravity-mode period spacing, and gravity-mode trapping in the isothermal upper layer.

## When to use

Use when a computed wave eigenproblem must be checked against theoretical spectral regularities — constant frequency spacing of acoustic modes, constant period spacing of gravity modes — or when mode trapping regions must be visualized as eigenfrequency vs. horizontal wavenumber.

## When NOT to use

- The problem is a time-dependent initial-value simulation rather than a targeted sparse eigenvalue problem — use Dedalus's IVP solvers instead of solve_sparse.
- The eigenmodes cannot be organized into acoustic/gravity branches with identifiable frequencies near theoretical targets, since the spacing diagnostic requires mode identification per branch.
- Validation requires dense eigenvalue sweeps over all subproblems (e.g. small test problems like the Mathieu equation) rather than targeted sparse solves near a mode frequency.

## Inputs

- Radiative-atmosphere parameters for the plain-text NLBVP (Eddington radiative closure, Kramer-like opacity)
- Linear perturbation eigenproblem specification on a Legendre spectral basis
- Target eigenvalues near expected mode frequencies
- Grid of 20 horizontal wavenumbers

## Outputs

- Background atmosphere fields written to an HDF5 file (via h5py)
- Eigenvalues and eigenvectors per horizontal wavenumber from the sparse solves
- Acoustic-branch frequency spacings and gravity-branch period spacings (numpy arrays)
- Eigenspectrum plot (eigenfrequency vs. horizontal wavenumber) with acoustic and gravity branches marked

## How to apply

First build the static atmosphere by formulating the plain-text nonlinear boundary-value problem (NLBVP) with Eddington radiative closure and Kramer-like opacity, solving it via Newton iteration, and writing the background fields to an HDF5 file. Then set up the linear perturbation eigenproblem on a Legendre spectral basis for 20 horizontal wavenumbers, targeting eigenvalues near the expected mode frequencies, and solve each targeted sparse problem via Dedalus's solver.solve_sparse interface. Extract the eigenfrequencies and compute, with numpy, the acoustic-branch frequency spacing and gravity-branch period spacing across the 20 wavenumbers, where the validation expectation is approximately constant spacing along each branch. Finally plot eigenfrequency vs. horizontal wavenumber with matplotlib, marking the acoustic and gravity branches, and confirm gravity modes are confined to the isothermal upper layer as in the paper's reported spectrum.

## Related tools

- **Dedalus** (Formulates the plain-text NLBVP and linear eigenproblem and performs the targeted sparse eigenvalue solves (solver.solve_sparse) per horizontal wavenumber) — https://github.com/DedalusProject/dedalus
- **numpy** (Extracts eigenfrequencies and computes the acoustic frequency spacings and gravity period spacings across the 20 wavenumbers)
- **matplotlib** (Plots the eigenspectrum (eigenfrequency vs. horizontal wavenumber), marking the acoustic and gravity branches)
- **h5py** (Writes the background atmosphere fields from the Newton-iterated NLBVP solution to an HDF5 file for reuse in the eigenproblem) — https://github.com/h5py/h5py
- **scipy** (Supporting numerical routines for the eigenproblem setup in the Dedalus Python environment) — https://github.com/scipy/scipy

## Examples

```
import dedalus.public as d3; solver.solve_sparse(solver.subproblems[1], NEV, target=target)
```

## Evaluation signals

- Acoustic (sound-wave) modes at fixed horizontal wavenumber show approximately constant frequency spacing across the branch.
- Gravity modes show approximately constant period spacing across the 20 horizontal wavenumbers.
- Gravity modes are spatially confined to the isothermal upper layer of the solved atmosphere, matching the qualitative spectrum in Figs. 15-17.
- Each per-wavenumber sparse solve converges to eigenvalues near the targeted expected mode frequencies.
- The Newton iteration on the NLBVP converges and the background fields are successfully written to the HDF5 file.

## Limitations

- Eigenvalue targeting is needed: sparse solves must be directed near expected mode frequencies per wavenumber, so a bad target can miss the branch.
- Spacings are only approximately constant; deviations from perfect regularity are expected, so validation is qualitative rather than exact.
- The skill depends on the bundled atmospheric-waves example workflow (dedalus/src/dedalus/examples) and on the installed Dedalus/Python environment.

## Evidence

- [abstract] Illustrative examples include atmospheric waves: "They also illustrate its capabilities with diverse examples, including optical network dynamics, magnetized shocks in plasmas, large-scale oceanic flows, low Reynolds number flows, stellar and"
- [results] Sparse targeted eigenvalue solves are the core mechanism: "solver.solve_sparse(solver.subproblems[1], NEV, target=target)"
- [results] Eigenvalue results are validated against reference values: "The eigenvalues are compared to the results of Vasil et al. (2016) [1] in Table 3."
- [abstract] Plain-text equations become efficient spectral solvers: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
