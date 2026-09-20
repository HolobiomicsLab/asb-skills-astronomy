---
name: legendre-spectral-basis-eigenvalue-problem-setup
description: Use when a linearized PDE (e.g., wave perturbations on a static atmosphere)
  must be posed as an eigenvalue problem over multiple horizontal wavenumbers and
  solved with targeted sparse eigenvalue extraction in Dedalus.
when_to_use_negative:
- The problem is a full nonlinear time integration rather than a linear eigenvalue
  problem — Dedalus IVP solvers, not solve_sparse, are appropriate
- The domain geometry or physics calls for a Fourier/ballistic basis rather than a
  Legendre spectral basis (e.g., purely periodic 1D problems like the paper's Mathieu
  example solved with a Fourier basis)
- Dense spectrum information over all eigenvalues is needed for a small matrix — use
  solver.solve_dense on a single subproblem instead of many targeted sparse solves
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Dedalus
  role: 'Core framework: Legendre spectral basis, symbolic equation entry, and the
    sparse eigenvalue interface (solver.solve_sparse)'
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Collect and array-ify eigenvalues across the 20 horizontal wavenumbers for
    spacing analysis
  repo: https://github.com/scipy/scipy
- name: h5py
  role: Read/write the background NLBVP fields consumed by the eigenproblem setup
  repo: https://github.com/h5py/h5py
- name: matplotlib
  role: Plot the eigenspectrum (eigenfrequency vs. horizontal wavenumber) marking
    acoustic and gravity branches
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:d417acc919f0e526bbccd33534d168d979f6304cdc91401b158980c28e4920bb
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Legendre spectral basis eigenvalue problem setup

## Summary

Set up linear perturbation eigenvalue problems on a Legendre spectral basis in Dedalus and solve them with targeted sparse eigensolves across a range of horizontal wavenumbers, as demonstrated for atmospheric acoustic and gravity wave modes.

## When to use

Use when a linearized PDE (e.g., wave perturbations on a static atmosphere) must be posed as an eigenvalue problem over multiple horizontal wavenumbers and solved with targeted sparse eigenvalue extraction in Dedalus.

## When NOT to use

- The problem is a full nonlinear time integration rather than a linear eigenvalue problem — Dedalus IVP solvers, not solve_sparse, are appropriate
- The domain geometry or physics calls for a Fourier/ballistic basis rather than a Legendre spectral basis (e.g., purely periodic 1D problems like the paper's Mathieu example solved with a Fourier basis)
- Dense spectrum information over all eigenvalues is needed for a small matrix — use solver.solve_dense on a single subproblem instead of many targeted sparse solves

## Inputs

- Dedalus problem/basis object with a Legendre spectral basis and symbolic equation specification
- Background state fields from the solved NLBVP (written to an HDF5 file via h5py)
- Set of horizontal wavenumbers (e.g., 20 values)
- Target eigenvalues near expected mode frequencies and NEV (number of eigenvalues) parameter

## Outputs

- Eigenvalues (growth rates/frequencies) per horizontal wavenumber
- Eigenvectors in the spectral coefficient representation per branch (acoustic and gravity)
- Arrays ready for eigenspectrum plotting (eigenfrequency vs. horizontal wavenumber)

## How to apply

First build and solve the static background (e.g., the radiative-atmosphere NLBVP with Eddington radiative closure and Kramer-like opacity via Newton iteration, writing background fields to HDF5 with h5py), since the eigenproblem depends on those fields. Then formulate the linear perturbation equations in Dedalus's plain-text symbolic form on a Legendre spectral basis. Loop over the desired horizontal wavenumbers (e.g., 20) and, for each, call the sparse solver with a target eigenvalue near the expected mode frequency: solver.solve_sparse(solver.subproblems[1], NEV, target=target). Targeting near expected frequencies keeps the returned eigenvectors on the physical acoustic/gravity branches instead of spurious spectral branches. Collect eigenvalues per wavenumber into numpy arrays for subsequent spacing analysis.

## Related tools

- **Dedalus** (Core framework: Legendre spectral basis, symbolic equation entry, and the sparse eigenvalue interface (solver.solve_sparse)) — https://github.com/DedalusProject/dedalus
- **numpy** (Collect and array-ify eigenvalues across the 20 horizontal wavenumbers for spacing analysis) — https://github.com/scipy/scipy
- **h5py** (Read/write the background NLBVP fields consumed by the eigenproblem setup) — https://github.com/h5py/h5py
- **matplotlib** (Plot the eigenspectrum (eigenfrequency vs. horizontal wavenumber) marking acoustic and gravity branches)

## Examples

```
solver.solve_sparse(solver.subproblems[1], NEV, target=target)
```

## Evaluation signals

- For each wavenumber, the sparse solve returns NEV eigenvalues near the specified target without convergence failures
- Acoustic branch shows approximately constant frequency spacing between successive modes at fixed horizontal wavenumber
- Gravity branch shows approximately constant period spacing between successive modes
- Gravity modes are spatially confined to the isothermal upper layer of the background atmosphere
- Sanity check the solve interface by reproducing the paper's simpler dense case (e.g., waves-on-a-string eigenvalues vs. (n*pi/Lx)**2 with small relative error)

## Limitations

- Targeted sparse solves return only eigenvalues near the target; spurious or missed branches can appear if the target is far from expected mode frequencies
- Results depend on a correct background state — a poorly converged Newton iteration of the NLBVP propagates into wrong eigenmodes
- Eigenproblem results are sensitive to the Legendre basis resolution and the number of horizontal wavenumbers sampled

## Evidence

- [intro] symbolic vectorial equation specification and sparse solve interface: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [results] targeted sparse eigensolve call: [evidence span withheld — anchor status: not_found]
- [results] sparse solve invocation pattern: "solver.solve_sparse(solver.subproblems[1], NEV, target=target)"
- [abstract] plain-text equations translated to spectral solvers: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [abstract] atmospheric waves example availability: "They also illustrate its capabilities with diverse examples, including optical network dynamics, magnetized shocks in plasmas, large-scale oceanic flows, low Reynolds number flows, stellar and"
