---
name: sparse-targeted-eigenvalue-solve
description: Use when you need targeted eigenvalues or eigenmodes of a linear PDE
  discretized on a spectral basis, swept over control parameters such as horizontal
  wavenumber, and the dense spectrum is too large or you only want modes near a known
  frequency/growth-rate target.
when_to_use_negative:
- The operator is small enough to diagonalize directly — the paper's Mathieu and waves-on-a-string
  examples use solver.solve_dense(solver.subproblems[0], rebuild_matrices=True) to
  get the full spectrum and compare against analytic values
- You need a time-dependent initial-value simulation of the PDE rather than a linear
  eigenmode analysis
- The problem is not discretized through Dedalus's spectral problem/subproblem interface,
  since solve_sparse operates on Dedalus subproblem objects
tools:
- name: Dedalus
  role: Provides the sparse eigenvalue solver interface (solver.solve_sparse) operating
    on subproblems of the discretized linear PDE
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Numerical processing of extracted eigenvalues and computation of mode spacings
    across the parameter sweep
- name: matplotlib
  role: Plotting the eigenspectrum (eigenfrequency vs. horizontal wavenumber) with
    acoustic and gravity branches marked
- name: h5py
  role: Reading/writing the static background fields (e.g. the radiative-atmosphere
    NLBVP solution) used to linearize the eigenproblem
  repo: https://github.com/h5py/h5py
- name: scipy
  role: Supporting special functions (e.g. from scipy.special import jv) used in formulating
    example problems
  repo: https://github.com/scipy/scipy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:85d06d754619c9ef7f1612dbe74e917a57b22fb7e04893ec5514017ed7ec9aad
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# sparse-targeted-eigenvalue-solve

## Summary

Solve linear eigenvalue problems for a discretized PDE operator with Dedalus's sparse solver, targeting a chosen eigenvalue region and sweeping a control parameter (e.g. horizontal wavenumber) to build mode spectra. This is the standard route for computing growth rates and wave eigenmodes (acoustic/gravity branches) in Dedalus applications.

## When to use

Use when you need targeted eigenvalues or eigenmodes of a linear PDE discretized on a spectral basis, swept over control parameters such as horizontal wavenumber, and the dense spectrum is too large or you only want modes near a known frequency/growth-rate target.

## When NOT to use

- The operator is small enough to diagonalize directly — the paper's Mathieu and waves-on-a-string examples use solver.solve_dense(solver.subproblems[0], rebuild_matrices=True) to get the full spectrum and compare against analytic values
- You need a time-dependent initial-value simulation of the PDE rather than a linear eigenmode analysis
- The problem is not discretized through Dedalus's spectral problem/subproblem interface, since solve_sparse operates on Dedalus subproblem objects

## Inputs

- Dedalus linear eigenproblem (problem object with discretized spectral basis, e.g. Legendre basis)
- Solved background/static state (e.g. radiative-atmosphere NLBVP background written to HDF5 via h5py)
- Sweep of control-parameter values (e.g. 20 horizontal wavenumbers)
- Target eigenvalue (expected mode frequency/growth rate) and NEV (number of requested eigenvalues)
- Relevant subproblem handle from solver.subproblems

## Outputs

- Eigenvalues (growth rates or eigenfrequencies) per control-parameter value
- Eigenvectors (mode functions) for each targeted eigenvalue
- Mode-spacing diagnostics (acoustic frequency spacing, gravity-wave period spacing)
- Eigenspectrum plot (eigenfrequency vs. horizontal wavenumber) with marked branches

## How to apply

First set up the linear perturbation problem in Dedalus on a spectral basis (e.g. a Legendre basis over the physical domain), either around a prescribed static background (such as the radiative-atmosphere NLBVP state written to HDF5) or a linearized channel/pipe flow. Identify the relevant subproblem from solver.subproblems for the current basis configuration, choose a target eigenvalue near the expected mode frequencies, and request a fixed number of eigenvalues (NEV) with solver.solve_sparse rather than solve_dense — dense solves are reserved for small problems like the Mathieu example where the full spectrum is affordable. Repeat the targeted sparse solve across the parameter sweep (e.g. 20 horizontal wavenumbers), collecting eigenvalues and eigenvectors per parameter value. Then inspect branch structure: compute frequency spacings on the acoustic branch and period spacings on the gravity branch, or compare growth rates against reference values (e.g. Rayleigh-Benard growth rates at Rayleigh = 1710, Prandtl = 1). The rationale for targeting is that sparse shift-invert solves return only modes near the target, which is efficient when you care about a specific physical branch rather than the whole spectrum.

## Related tools

- **Dedalus** (Provides the sparse eigenvalue solver interface (solver.solve_sparse) operating on subproblems of the discretized linear PDE) — https://github.com/DedalusProject/dedalus
- **numpy** (Numerical processing of extracted eigenvalues and computation of mode spacings across the parameter sweep)
- **matplotlib** (Plotting the eigenspectrum (eigenfrequency vs. horizontal wavenumber) with acoustic and gravity branches marked)
- **h5py** (Reading/writing the static background fields (e.g. the radiative-atmosphere NLBVP solution) used to linearize the eigenproblem) — https://github.com/h5py/h5py
- **scipy** (Supporting special functions (e.g. from scipy.special import jv) used in formulating example problems) — https://github.com/scipy/scipy

## Examples

```
solver.solve_sparse(solver.subproblems[1], NEV, target=target)
```

## Evaluation signals

- Targeted sparse solve returns NEV eigenvalues clustered near the chosen target, consistent with the physics (e.g. Rayleigh-Benard growth rates at Rayleigh = 1710, Prandtl = 1)
- For problems with known analytic eigenvalues (e.g. waves on a string, true_evals = (n * np.pi / Lx)**2), relative errors between computed and true eigenvalues are small
- Acoustic branch exhibits approximately constant frequency spacing at fixed horizontal wavenumber
- Gravity branch exhibits approximately constant period spacing, with gravity modes confined to the isothermal upper layer of the atmosphere
- The sparse solve on the correct subproblem (solver.subproblems index matching the basis configuration) reproduces the dense-solve eigenvalues for small test cases like the Mathieu problem

## Limitations

- Sparse solves are targeted: modes far from the chosen target are not returned, so a poor target choice can miss physical branches
- The correct subproblem index must be selected from solver.subproblems; using the wrong one yields incorrect eigenpairs
- Convergence and spectral accuracy depend on the spectral basis resolution and the quality of the static background state

## Evidence

- [results] Sparse targeted solve is the documented interface for parameter-swept eigenproblems: [evidence span withheld — anchor status: not_found]
- [results] Concrete sparse solve call signature on a subproblem with target: "solver.solve_sparse(solver.subproblems[1], NEV, target=target)"
- [results] Dense solves are used for small problems like the Mathieu eigenvalue problem: [evidence span withheld — anchor status: not_found]
- [intro] Dedalus's purpose as a spectral PDE framework grounding the eigenproblem setup: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [results] Analytic eigenvalue validation for waves on a string: "true_evals = (n * np.pi / Lx)**2"
