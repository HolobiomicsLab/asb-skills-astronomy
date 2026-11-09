---
name: newton-iteration-nlbvp-static-atmosphere-solve
description: Use when you need a stratified background atmosphere (e.g. radiative
  layer with Eddington closure and Kramer-like opacity) as the equilibrium state for
  a linear perturbation or eigenvalue problem in Dedalus.
when_to_use_negative:
- The problem is already linear — a direct sparse/dense eigenvalue solve (solver.solve_sparse
  / solve_dense) is the appropriate route instead of a Newton NLBVP
- The background atmosphere is analytic or isothermal and can be specified in closed
  form, so no non-linear boundary value problem needs to be solved
- You need fully dynamical (time-dependent) atmosphere evolution rather than a static
  equilibrium — an IVP simulation, not an NLBVP, is required
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3315
tools:
- name: Dedalus
  role: Formulates the plain-text NLBVP, discretizes it with global spectral methods,
    and runs the Newton iteration for the static atmosphere
  repo: https://github.com/DedalusProject/dedalus
- name: h5py
  role: Writes the converged background fields to an HDF5 file consumed by the eigenproblem
    stage
  repo: https://github.com/h5py/h5py
- name: numpy
  role: Array handling for background fields and grids used in the Newton solve
  repo: https://github.com/scipy/scipy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:ec38d1eda571e1cd8031bdbe48fecff9b738142ca5b8db5628062e4b9bed2e21
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# newton-iteration-nlbvp-static-atmosphere-solve

## Summary

Solve the static stellar/radiative atmosphere as a non-linear boundary value problem (NLBVP) formulated in plain-text equations inside Dedalus, using Newton iteration to converge the background state (density, temperature, opacity) that is then written to HDF5 for subsequent wave-eigenmode analysis.

## When to use

Use when you need a stratified background atmosphere (e.g. radiative layer with Eddington closure and Kramer-like opacity) as the equilibrium state for a linear perturbation or eigenvalue problem in Dedalus.

## When NOT to use

- The problem is already linear — a direct sparse/dense eigenvalue solve (solver.solve_sparse / solve_dense) is the appropriate route instead of a Newton NLBVP
- The background atmosphere is analytic or isothermal and can be specified in closed form, so no non-linear boundary value problem needs to be solved
- You need fully dynamical (time-dependent) atmosphere evolution rather than a static equilibrium — an IVP simulation, not an NLBVP, is required

## Inputs

- Plain-text NLBVP equation system (hydrostatic balance, radiative transfer with Eddington closure, Kramer-like opacity) expressed in Dedalus symbolic form
- Boundary conditions for the static atmosphere (e.g. temperature/optical-depth conditions at top and bottom)
- Spectral domain/basis definition for the 1-D radial atmosphere

## Outputs

- Converged 1-D background atmosphere fields (temperature, density, pressure profiles) from the Newton iteration
- HDF5 file of the static atmosphere background, loadable by the downstream linear eigenproblem solver

## How to apply

Formulate the hydrostatic atmosphere as a plain-text NLBVP in Dedalus, embedding the Eddington radiative closure and Kramer-like opacity directly in the equation strings so Dedalus translates them into spectral discretizations. Iterate with Newton's method until the residual of the non-linear system converges, checking that the background satisfies hydrostatic balance and radiative equilibrium at the boundaries. Write the converged background fields (temperature, density, pressure) to an HDF5 file with h5py so the downstream linear eigenproblem can load the static state. This is the first stage of the atmospheric-waves example (Figs. 15–17 of the paper), where the resulting background feeds a Legendre-basis eigenproblem over 20 horizontal wavenumbers; the static solve must be accurate since gravity-mode confinement to the isothermal upper layer depends on the stratification.

## Related tools

- **Dedalus** (Formulates the plain-text NLBVP, discretizes it with global spectral methods, and runs the Newton iteration for the static atmosphere) — https://github.com/DedalusProject/dedalus
- **h5py** (Writes the converged background fields to an HDF5 file consumed by the eigenproblem stage) — https://github.com/h5py/h5py
- **numpy** (Array handling for background fields and grids used in the Newton solve) — https://github.com/scipy/scipy

## Examples

```
python3 -m dedalus test  # verify build, then run bundled atmospheric-waves example from dedalus/src/dedalus/examples to NLBVP-converge the radiative atmosphere and write the HDF5 background
```

## Evaluation signals

- Newton iteration residual for the NLBVP decreases to a tight tolerance (converged background state)
- Resulting background satisfies hydrostatic balance and radiative equilibrium as prescribed by the plain-text equation system
- Background fields written to HDF5 are loadable by the subsequent linear perturbation/eigenproblem stage without re-derivation
- Downstream wave spectrum shows gravity modes confined to the isothermal upper layer, a qualitative check that the stratification is correct

## Limitations

- Convergence of Newton iteration is not guaranteed for arbitrary opacity/closure choices; poor initial guesses or stiff stratifications may need continuation or relaxation
- The static solve is a prerequisite for, but says nothing about, the eigenmode spectrum itself — errors in the background propagate to mode frequencies and spacings
- Dedalus installation is non-trivial (requires building against BLAS/OpenBLAS, FFTW, HDF5 with optional MPI support); a broken build invalidates the NLBVP solve

## Evidence

- [abstract] Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods.: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [abstract] Diverse examples including stellar and atmospheric waves: "They also illustrate its capabilities with diverse examples, including optical network dynamics, magnetized shocks in plasmas, large-scale oceanic flows, low Reynolds number flows, stellar and"
- [intro] Easy-to-use symbolic interface: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [results] Sparse eigenvalue solve precedent in paper's examples: [evidence span withheld — anchor status: not_found]
- [intro] h5py usage in Dedalus: "import h5py
import numpy as np"
