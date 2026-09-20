---
name: chebyshev-spectral-discretization
description: Use when discretizing a 1D PDE on [-1, 1] with a Chebyshev T_n expansion
  and banded Dirichlet/Neumann boundary conditions, and you need a sparse, banded
  pencil matrix assembled stage-by-stage (tau rows, recombination, mode interleaving)
  rather than a dense Chebyshev collocation matrix.
when_to_use_negative:
- The problem needs dense eigensolves of a small transformed operator (as in the paper's
  Mathieu example solved with solve_dense), where banded sparsity gains are irrelevant.
- The PDE is posed on a non-Chebyshev-friendly domain or with non-polynomial boundary
  conditions that cannot be expressed as tau boundary rows on coefficient coefficients.
- You only need Dedalus's ready-made equation parser rather than reconstructing the
  matrix pipeline itself; building operators by hand duplicates Dedalus internals.
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Dedalus
  role: Framework whose sparse tau-method matrix pipeline (first-order system, T-to-U
    conversion, Dirichlet preconditioning, mode interleaving) is reconstructed; also
    usable directly for dense or sparse solves via solver.solve_dense/solve_sparse.
  repo: https://github.com/DedalusProject/dedalus
- name: numpy
  role: Constructs the Chebyshev T_n basis, second-derivative operator, conversion
    operators, and stores the staged banded matrices in the .npz file.
- name: scipy
  role: Sparse-matrix assembly and verification of banded structure; supplies sparse
    solvers alongside Dedalus.
  repo: https://github.com/scipy/scipy
- name: matplotlib
  role: Visualization of the staged matrix sparsity patterns for comparison against
    the Fig. 3 layout.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:b954b1f338ab8b0ba19481b7203e2092876b9d31135ddd90fda0f785f8531cd0
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# chebyshev-spectral-discretization

## Summary

Constructs sparse, fully banded spectral discretizations for 1D PDEs (e.g., the Chebyshev Poisson problem of Eq. 19) by combining the sparse tau method: first-order system formulation, T-to-U test-basis conversion, Dirichlet preconditioning via basis recombination, and mode-interleaved assembly of the pencil matrix as staged in Fig. 3.

## When to use

Use when discretizing a 1D PDE on [-1, 1] with a Chebyshev T_n expansion and banded Dirichlet/Neumann boundary conditions, and you need a sparse, banded pencil matrix assembled stage-by-stage (tau rows, recombination, mode interleaving) rather than a dense Chebyshev collocation matrix.

## When NOT to use

- The problem needs dense eigensolves of a small transformed operator (as in the paper's Mathieu example solved with solve_dense), where banded sparsity gains are irrelevant.
- The PDE is posed on a non-Chebyshev-friendly domain or with non-polynomial boundary conditions that cannot be expressed as tau boundary rows on coefficient coefficients.
- You only need Dedalus's ready-made equation parser rather than reconstructing the matrix pipeline itself; building operators by hand duplicates Dedalus internals.

## Inputs

- 1D PDE specification (e.g., Poisson problem of Eq. 19) on the domain x in [-1, 1]
- Polynomial order N for the Chebyshev T_n expansion basis
- Boundary condition operators (Dirichlet u(±1)=0; Neumann counterparts)
- T-to-U (ultraspherical) test-basis conversion relations

## Outputs

- Staged matrices: pre-conversion, tau-augmented, recombined, and mode-interleaved pencil
- Final sparse, fully banded pencil matrix matching the Fig. 3 layout
- Bandedness summary (bandwidth and nonzero density) stored in poisson_staged_matrices.npz

## How to apply

Build the Chebyshev T_n expansion basis and second-derivative operator in numpy for the 1D Poisson problem of Eq. (19) on x in [-1, 1]. Convert derivative terms into the U (ultraspherical) test basis using the T-to-U conversion operator so the resulting derivative matrices are banded rather than dense (the tau method). Append tau boundary rows enforcing the Dirichlet conditions u(±1)=0 in place of the highest-order coefficient rows, then apply Dirichlet basis recombination so the retained variable modes exactly satisfy the boundary conditions, producing the recombined stage matrix. Finally, interleave equation rows and variable columns by mode index (not by variable block) to match the staged pencil layout of Fig. 3, since the fully banded structure depends on combining all four mechanisms: first-order system formulation, T-to-U mapping, preconditioning, and mode-level interleaving.

## Related tools

- **Dedalus** (Framework whose sparse tau-method matrix pipeline (first-order system, T-to-U conversion, Dirichlet preconditioning, mode interleaving) is reconstructed; also usable directly for dense or sparse solves via solver.solve_dense/solve_sparse.) — https://github.com/DedalusProject/dedalus
- **numpy** (Constructs the Chebyshev T_n basis, second-derivative operator, conversion operators, and stores the staged banded matrices in the .npz file.)
- **scipy** (Sparse-matrix assembly and verification of banded structure; supplies sparse solvers alongside Dedalus.) — https://github.com/scipy/scipy
- **matplotlib** (Visualization of the staged matrix sparsity patterns for comparison against the Fig. 3 layout.)

## Examples

```
import numpy as np
import dedalus.public as d3
```

## Evaluation signals

- All nonzeros of the final pencil lie within a fixed bandwidth around the diagonal (verify against the stored bandedness summary).
- Nonzero density of the assembled matrix is low (sparse), contrasting with a dense Chebyshev collocation second-derivative matrix.
- The staged pipeline (pre-conversion → tau-augmented → recombined → interleaved) reproduces the layout of Fig. 3 row-by-row.
- Solve or eigen-analyze the assembled system and confirm results match reference values, as the paper checks eigenvalues against analytic values like (n * np.pi / Lx)**2 or against Vasil et al. (2016) Table 3.
- Dirichlet rows exactly enforce u(±1)=0 after recombination: apply the recombined stage to a trial expansion and evaluate at the endpoints.

## Limitations

- The fully banded structure only emerges when ALL four mechanisms are combined (first-order system, T-to-U mapping, Dirichlet preconditioning, mode interleaving); omitting any one yields a non-banded or incorrectly conditioned pencil.
- Dedalus installation can be involved; the paper's methods describe building from source (pip3 install -r requirements.txt; python3 setup.py build_ext --inplace) with dependencies on OpenBLAS, HDF5 with parallel MPI support, and FFTW built with MPI/threads.
- No changelog tracked for the associated tooling context; users should pin versions when reproducing the staged matrices.

## Evidence

- [intro] Dedalus is a flexible framework for solving partial differential equations using modern spectral methods.: "Dedalus is a flexible framework for solving partial differential equations using modern spectral methods."
- [intro] Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification.: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [results] Solved sparse eigenvalue problems for Rayleigh-Benard growth rates over a range of horizontal wavenumbers: [evidence span withheld — anchor status: not_found]
- [readme] Summary: A flexible framework for solving PDEs with modern spectral methods.: [evidence span withheld — anchor status: external_doc]
