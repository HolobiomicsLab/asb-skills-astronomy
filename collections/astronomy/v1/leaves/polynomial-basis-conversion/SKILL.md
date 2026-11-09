---
name: polynomial-basis-conversion
description: Use when a PDE discretization in a Chebyshev T_n expansion must be re-expressed
  in the U (ultraspherical) test basis via T-to-U conversion, with tau boundary rows
  and basis recombination producing banded pencil matrices.
when_to_use_negative:
- The discretization already uses an ultraspherical/U test basis, so no T-to-U conversion
  step is needed
- The problem has non-Dirichlet or mixed boundary conditions incompatible with the
  Dirichlet basis recombination rows described here
- A dense spectral Galerkin formulation is acceptable and banded sparsity structure
  is not a requirement
tools:
- name: Dedalus
  role: Reference implementation of the sparse tau method whose staged matrix construction
    (first-order system, T-to-U conversion, Dirichlet preconditioning, mode interleaving)
    this skill reproduces
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Constructs the Chebyshev T_n basis, T-to-U conversion operator, tau-augmented
    and recombined banded matrices, and saves the staged npz archive
- name: matplotlib
  role: Visualizes the sparsity/band structure of the staged matrices for validation
    against the Fig. 3 layout
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:776a186ecb80978b2bf18fec7b6bd13bff662e0cc355a6b0d91b72ab0a454370
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DedalusProject/dedalus
  tools:
  - Dedalus
  - Python
  - matplotlib
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# polynomial-basis-conversion

## Summary

Reconstructs Dedalus's sparse tau-method pipeline for a 1D Chebyshev Poisson problem (Eq. 19), converting derivative operators from the Chebyshev T basis into the U (ultraspherical) test basis and staging the fully banded pencil matrix exactly as in Fig. 3.

## When to use

Use when a PDE discretization in a Chebyshev T_n expansion must be re-expressed in the U (ultraspherical) test basis via T-to-U conversion, with tau boundary rows and basis recombination producing banded pencil matrices.

## When NOT to use

- The discretization already uses an ultraspherical/U test basis, so no T-to-U conversion step is needed
- The problem has non-Dirichlet or mixed boundary conditions incompatible with the Dirichlet basis recombination rows described here
- A dense spectral Galerkin formulation is acceptable and banded sparsity structure is not a requirement

## Inputs

- 1D Chebyshev Poisson problem definition (Eq. 19) with Dirichlet conditions u(±1)=0 on x in [-1, 1]
- Polynomial cutoff N (number of Chebyshev T_n modes)
- Second-derivative operator in the Chebyshev T_n expansion basis

## Outputs

- poisson_staged_matrices.npz storing pre-conversion, tau-augmented, recombined, and interleaved matrices
- Bandedness summary (bandwidth, nonzero density) for the final assembled pencil
- Final sparse, fully banded pencil matrix with mode-interleaved ordering as in Fig. 3

## How to apply

In Python with numpy, first construct the Chebyshev T_n expansion basis and the second-derivative operator for the 1D Poisson problem of Eq. (19) on x in [-1, 1]. Build the T-to-U test-basis conversion operator so derivative terms are expressed in the U basis, following the paper's conversion relations. Append tau boundary rows enforcing the Dirichlet conditions u(±1)=0 in place of the highest-order coefficient rows, then apply Dirichlet basis recombination so retained variable modes exactly satisfy the boundary conditions. Finally assemble the pencil with mode-interleaved row/column ordering (equation rows and variable columns alternating by mode index) matching the staged layout of Fig. 3; this interleaving, together with first-order system formulation and conversion, is what yields the fully banded structure.

## Related tools

- **Dedalus** (Reference implementation of the sparse tau method whose staged matrix construction (first-order system, T-to-U conversion, Dirichlet preconditioning, mode interleaving) this skill reproduces) — https://github.com/DedalusProject/dedalus
- **numpy** (Constructs the Chebyshev T_n basis, T-to-U conversion operator, tau-augmented and recombined banded matrices, and saves the staged npz archive)
- **matplotlib** (Visualizes the sparsity/band structure of the staged matrices for validation against the Fig. 3 layout)

## Examples

```
import numpy as np; np.savez('poisson_staged_matrices.npz', pre_conversion=A0, tau_augmented=A1, recombined=A2, interleaved=A3, bandedness=summary)
```

## Evaluation signals

- All nonzero entries of the final pencil lie within a fixed bandwidth around the diagonal (bandedness check passes)
- Nonzero density of the assembled matrix is low (sparse), consistent with the fully banded result of Fig. 3
- Retained variable modes of the recombined matrix exactly satisfy u(±1)=0 (Dirichlet boundary conditions)
- All four staged matrices (pre-conversion, tau-augmented, recombined, interleaved) are present in poisson_staged_matrices.npz
- Solving the assembled pencil recovers the Poisson solution for a manufactured right-hand side

## Limitations

- Breaks the banded structure if rows/columns are ordered by variable rather than interleaved by mode index — interleaving by mode is essential to match the Fig. 3 layout
- Dirichlet preconditioning/basis recombination described here is specific to boundary rows enforcing Dirichlet conditions; other boundary types require different recombination
- Derived for a 1D Chebyshev Poisson problem (Eq. 19); extension to multidimensional or coupled systems requires reformulating as a first-order system as Dedalus does

## Evidence

- [intro] Spectral-method framework: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [abstract] Global spectral solver translation: "Dedalus translates plain-text equations into efficient and parallelized solvers using global spectral methods."
- [intro] Python implementation basis: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [readme] Framework scope from README: [evidence span withheld — anchor status: not_found]
