---
name: atmospheric-wave-eigenvalue-analysis
description: Use when reproducing or validating stellar/atmospheric wave eigenmode
  spectra from a stratified atmosphere, checking acoustic frequency spacing and gravity-mode
  period spacing against theory.
when_to_use_negative:
- When the goal is nonlinear time integration of turbulence or initial-value problems
  rather than a linear, targeted eigenvalue (NLBVP + sparse eigenmode) analysis
- When the background stratification is fully prescribed (e.g. isothermal only) and
  no radiative-atmosphere NLBVP needs to be solved
- When the analysis target is dense eigenvalue solves on a single subproblem (e.g.
  Mathieu problem) instead of targeted sparse solves across many wavenumbers
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Dedalus
  role: Symbolic plain-text equation specification, NLBVP Newton solve, and targeted
    sparse eigenvalue solves via solver.solve_sparse
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Extract and compute acoustic frequency spacings and gravity-mode period spacings
    across the 20 wavenumbers
- name: matplotlib
  role: Plot the eigenspectrum (eigenfrequency vs. horizontal wavenumber), marking
    acoustic and gravity branches
- name: h5py
  role: Write and read the background atmosphere fields to HDF5
  repo: https://github.com/h5py/h5py
- name: scipy
  role: Supporting numerical routines alongside Dedalus in the example scripts
  repo: https://github.com/scipy/scipy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:35de8cb489a5b128901870c177767ff4a6949f79a41c161b5f91fb50f46cc1d2
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# atmospheric-wave-eigenvalue-analysis

## Summary

Reproduces the Dedalus radiative-atmosphere example: solve the static NLBVP atmosphere (Eddington radiative closure, Kramer-like opacity), then compute linear wave eigenmodes over 20 horizontal wavenumbers to verify the expected spectral regularities — constant frequency spacing for acoustic (sound) modes and constant period spacing for gravity modes.

## When to use

Use when reproducing or validating stellar/atmospheric wave eigenmode spectra from a stratified atmosphere, checking acoustic frequency spacing and gravity-mode period spacing against theory.

## When NOT to use

- When the goal is nonlinear time integration of turbulence or initial-value problems rather than a linear, targeted eigenvalue (NLBVP + sparse eigenmode) analysis
- When the background stratification is fully prescribed (e.g. isothermal only) and no radiative-atmosphere NLBVP needs to be solved
- When the analysis target is dense eigenvalue solves on a single subproblem (e.g. Mathieu problem) instead of targeted sparse solves across many wavenumbers

## Inputs

- Atmospheric waves example script from the bundled Dedalus examples (dedalus/src/dedalus/examples)
- NLBVP problem specification as plain-text vectorial equations (Eddington radiative closure, Kramer-like opacity)
- 20 target horizontal wavenumbers and expected target mode frequencies

## Outputs

- HDF5 file of static-atmosphere background fields (written with h5py)
- Eigenvalues/eigenvectors per horizontal wavenumber from sparse targeted solves
- Acoustic-branch frequency spacings and gravity-branch period spacings across the 20 wavenumbers
- Eigenspectrum plot (eigenfrequency vs. horizontal wavenumber) with acoustic and gravity branches marked

## How to apply

First locate and run the atmospheric waves example bundled in dedalus/src/dedalus/examples: build the static atmosphere as a plain-text NLBVP with Eddington radiative closure and Kramer-like opacity, solve it by Newton iteration, and write background fields to an HDF5 file via h5py. Then formulate the linear perturbation eigenproblem on a Legendre spectral basis for 20 horizontal wavenumbers, targeting eigenvalues near the expected mode frequencies. Solve each targeted sparse eigenproblem with Dedalus's sparse solver interface (solver.solve_sparse) to get eigenvalues/eigenvectors per wavenumber. Finally use numpy to extract the acoustic-branch frequency spacing and gravity-branch period spacing across the 20 wavenumbers, and plot the eigenspectrum (eigenfrequency vs. horizontal wavenumber) with matplotlib, marking the acoustic and gravity branches.

## Related tools

- **Dedalus** (Symbolic plain-text equation specification, NLBVP Newton solve, and targeted sparse eigenvalue solves via solver.solve_sparse) — https://github.com/DedalusProject/dedalus
- **numpy** (Extract and compute acoustic frequency spacings and gravity-mode period spacings across the 20 wavenumbers)
- **matplotlib** (Plot the eigenspectrum (eigenfrequency vs. horizontal wavenumber), marking acoustic and gravity branches)
- **h5py** (Write and read the background atmosphere fields to HDF5) — https://github.com/h5py/h5py
- **scipy** (Supporting numerical routines alongside Dedalus in the example scripts) — https://github.com/scipy/scipy

## Examples

```
solver.solve_sparse(solver.subproblems[1], NEV, target=target)
```

## Evaluation signals

- Acoustic (sound) modes exhibit approximately constant frequency spacing at fixed horizontal wavenumber across all 20 wavenumbers
- Gravity modes exhibit approximately constant period spacing at fixed horizontal wavenumber
- Gravity modes are spatially confined to the isothermal upper layer of the atmosphere
- The computed eigenspectrum qualitatively matches the spectrum reported in the paper (Figs. 15-17)
- The NLBVP Newton iteration converges and the background fields round-trip through the HDF5 file without NaNs or discontinuities

## Limitations

- Sparse eigenvalue solves are targeted: only eigenvalues near the specified target mode frequencies are recovered, so choices of target and NEV control which modes appear
- The spectral regularities hold approximately, not exactly, and were verified only qualitatively against the paper's figures
- Gravity-mode confinement to the isothermal upper layer is a property of this particular radiative-atmosphere background and should be rechecked for other atmospheres

## Evidence

- [abstract] Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods.: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [results] Solved sparse eigenvalue problems for Rayleigh-Benard growth rates over a range of horizontal wavenumbers: [evidence span withheld — anchor status: not_found]
- [results] solver.solve_sparse(solver.subproblems[1], NEV, target=target): "solver.solve_sparse(solver.subproblems[1], NEV, target=target)"
- [readme] Summary: A flexible framework for solving PDEs with modern spectral methods.: [evidence span withheld — anchor status: external_doc]
