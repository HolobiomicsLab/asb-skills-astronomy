---
name: dirichlet-basis-recombination
description: Use when assembling sparse tau-method matrices for a 1D spectral PDE
  (e.g. Poisson with Dirichlet/Neumann BCs on x∈[-1,1]) and you need boundary conditions
  folded into the basis via recombination rather than ad-hoc constraint rows.
when_to_use_negative:
- The problem has no boundary conditions to encode in the basis (pure eigenvalue/periodic
  problems) — recombination rows would over-constrain the system.
- You need dense collocation-style matrices rather than banded pencil matrices; Dedalus's
  dense solve path (e.g. solve_dense for Mathieu eigenvalues) does not use this staged
  banded construction.
- Multi-dimensional problems where Chebyshev-U interleaving by mode does not directly
  apply without adaptation.
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: Dedalus
  role: Reference framework whose sparse tau-method pipeline (first-order system,
    T-to-U conversion, recombination, mode interleaving) is being reconstructed
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Constructs the Chebyshev T basis, derivative operators, conversion/recombination
    matrices, and saves staged matrices to npz
- name: scipy
  role: Sparse matrix storage and bandwidth/density checks on the assembled pencil
  repo: https://github.com/scipy/scipy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:99f5cd6de0e7654d89f20b8fab4c22b5926c8d5d983fa84a3527c86b21e040cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DedalusProject/dedalus
  tools:
  - Dedalus
  - Python
  - matplotlib
  - numpy
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# dirichlet-basis-recombination

## Summary

Reconstructs Dedalus's sparse tau-method matrix assembly for the 1D Chebyshev Poisson problem (Eq. 19): converting derivative terms from Chebyshev T to ultraspherical U test bases, appending tau boundary rows for u(±1)=0, applying Dirichlet basis recombination, and interleaving rows/columns by mode to yield a fully banded pencil like Fig. 3.

## When to use

Use when assembling sparse tau-method matrices for a 1D spectral PDE (e.g. Poisson with Dirichlet/Neumann BCs on x∈[-1,1]) and you need boundary conditions folded into the basis via recombination rather than ad-hoc constraint rows.

## When NOT to use

- The problem has no boundary conditions to encode in the basis (pure eigenvalue/periodic problems) — recombination rows would over-constrain the system.
- You need dense collocation-style matrices rather than banded pencil matrices; Dedalus's dense solve path (e.g. solve_dense for Mathieu eigenvalues) does not use this staged banded construction.
- Multi-dimensional problems where Chebyshev-U interleaving by mode does not directly apply without adaptation.

## Inputs

- 1D Poisson problem specification (Eq. 19: -u'' = f with Dirichlet/Neumann BCs on x ∈ [-1, 1])
- Chebyshev T_n expansion basis (numpy arrays of modal coefficients)
- Second-derivative operator on the T basis
- N (number of retained Chebyshev modes) controlling matrix size

## Outputs

- Pre-conversion derivative matrix (T-basis)
- T-to-U conversion operator and tau-augmented matrix with boundary rows
- Dirichlet-recombined stage matrix
- Mode-interleaved final pencil matrix (equation rows and variable columns alternating by mode index)
- Bandedness summary and all staged matrices in poisson_staged_matrices.npz

## How to apply

In Python/numpy, first build the Chebyshev T_n expansion basis and the second-derivative operator for Eq. (19), then form the T-to-U test-basis conversion operator so derivative terms live in the U basis, which is what keeps the derivative entries sparse. Append tau boundary rows enforcing u(±1)=0 in place of the highest-order coefficient rows, then apply Dirichlet basis recombination so the retained variable modes exactly satisfy the boundary conditions, producing the recombined stage matrix. Finally assemble the pencil with mode-interleaved ordering (equation rows and variable columns alternating by mode index) matching Fig. 3's staged layout; this interleaving-by-mode rather than by variable is one of the four mechanisms (first-order formulation, T-to-U conversion, recombination, interleaving) that make the matrix fully banded. Validate by checking all nonzeros lie within a fixed bandwidth around the diagonal and the density is low before persisting every stage.

## Related tools

- **Dedalus** (Reference framework whose sparse tau-method pipeline (first-order system, T-to-U conversion, recombination, mode interleaving) is being reconstructed) — https://github.com/DedalusProject/dedalus
- **numpy** (Constructs the Chebyshev T basis, derivative operators, conversion/recombination matrices, and saves staged matrices to npz)
- **scipy** (Sparse matrix storage and bandwidth/density checks on the assembled pencil) — https://github.com/scipy/scipy

## Examples

```
np.savez('poisson_staged_matrices.npz', pre_conversion=A_T, tau_augmented=A_tau, recombined=A_rc, interleaved=A_final, bandwidth=bandwidth)
```

## Evaluation signals

- Every nonzero entry of the final pencil lies within a fixed bandwidth around the diagonal (bandedness check passes)
- Nonzero density of the assembled pencil is low (sparse), matching the banded structure of Fig. 3
- Recombined basis modes satisfy u(±1)=0 exactly (Dirichlet conditions hold identically on the retained modes)
- All four staged matrices (pre-conversion, tau-augmented, recombined, interleaved) are present and shape-consistent in poisson_staged_matrices.npz
- Row/column ordering of the final pencil alternates equation rows and variable columns by mode index as in Fig. 3

## Limitations

- The construction is specific to 1D Chebyshev/Ultraspherical settings; extending to multidimensional bases requires re-deriving the interleaving.
- Incorrect T-to-U conversion breaks sparsity: derivative terms must be expressed in the U test basis following the paper's conversion relations, otherwise the pencil loses its banded structure.
- Tau boundary rows replace highest-order coefficient rows; appending them elsewhere (instead of basis recombination) yields a non-square or non-banded system.

## Evidence

- [abstract] framework scope: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [methods] sparse spectral machinery being reconstructed: "Dedalus is a flexible framework for solving partial differential equations using spectral methods."
- [results] sparse eigen-solve path analogous to staged banded matrices: [evidence span withheld — anchor status: not_found]
