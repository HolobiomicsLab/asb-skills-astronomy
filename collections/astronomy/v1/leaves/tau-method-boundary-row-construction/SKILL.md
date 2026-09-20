---
name: tau-method-boundary-row-construction
description: Use when reproducing or extending Dedalus-style sparse spectral pencils
  for a 1D boundary-value problem (e.g. Chebyshev Poisson on x in [-1,1]) where you
  need the staged tau-augmented, recombined, and interleaved banded matrices, not
  just a solution vector.
when_to_use_negative:
- The problem has no boundary conditions requiring tau rows or basis recombination
  (e.g. a pure spectral eigenvalue problem already solved densely, as in the paper's
  Mathieu example using solver.solve_dense)
- You only need PDE solutions or eigenvalues, not the underlying staged pencil matrices
  for method-level reconstruction
- The domain or basis is not the Chebyshev/Ultraspherical setting assumed by the T-to-U
  conversion relations
tools:
- name: Python
  role: implementation language for constructing the Chebyshev basis and staged matrices
- name: numpy
  role: builds the Chebyshev T_n expansion basis, derivative operators, and the staged
    banded matrices stored in .npz
- name: scipy
  role: sparse matrix representation to verify the final pencil is sparse and fully
    banded
  repo: https://github.com/scipy/scipy
- name: Dedalus
  role: reference framework whose sparse tau-method matrix construction is being reconstructed
  repo: https://github.com/DedalusProject/dedalus
- name: matplotlib
  role: visual inspection of the staged banded matrix structure (cf. Fig. 3 layout)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:0b700cd2be794896a69ac0541b4b7001918370083943b16be8fa9733c2064cd7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scipy/scipy
  tools:
  - Dedalus
  - Python
  - matplotlib
  - numpy
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# tau-method-boundary-row-construction

## Summary

Reconstructs Dedalus's sparse tau-method pipeline for assembling staged, fully banded pencil matrices for a 1D Chebyshev Poisson problem with Dirichlet conditions (Eq. 19, Fig. 3), combining first-order system formulation, T-to-U test-basis conversion, Dirichlet basis recombination, and mode-interleaved ordering.

## When to use

Use when reproducing or extending Dedalus-style sparse spectral pencils for a 1D boundary-value problem (e.g. Chebyshev Poisson on x in [-1,1]) where you need the staged tau-augmented, recombined, and interleaved banded matrices, not just a solution vector.

## When NOT to use

- The problem has no boundary conditions requiring tau rows or basis recombination (e.g. a pure spectral eigenvalue problem already solved densely, as in the paper's Mathieu example using solver.solve_dense)
- You only need PDE solutions or eigenvalues, not the underlying staged pencil matrices for method-level reconstruction
- The domain or basis is not the Chebyshev/Ultraspherical setting assumed by the T-to-U conversion relations

## Inputs

- PDE specification: 1D Chebyshev Poisson problem per Eq. (19) on x in [-1, 1] with Dirichlet boundary conditions u(±1)=0
- Chebyshev polynomial expansion basis (T_n) coefficients
- Numpy ndarray/sparse-matrix representations of the first- and second-derivative operators

## Outputs

- poisson_staged_matrices.npz containing pre-conversion, tau-augmented, recombined, and mode-interleaved staged matrices
- Bandedness summary (bandwidth around the diagonal and nonzero density) for the final assembled pencil

## How to apply

In Python with numpy, first build the Chebyshev T_n expansion basis and the second-derivative operator for the Poisson problem of Eq. (19) on x in [-1, 1]. Construct the T-to-U test-basis conversion operator so derivative terms are expressed in the U (ultraspherical) test basis, following the paper's conversion relations — this is what makes the derivative blocks sparse and banded. Append tau boundary rows enforcing the Dirichlet conditions u(±1)=0 in place of the highest-order coefficient rows, then apply Dirichlet basis recombination so retained variable modes exactly satisfy the boundary conditions, yielding the recombined stage matrix. Assemble the final pencil with mode-interleaved row/column ordering (equation rows and variable columns alternating by mode index) matching the staged layout of Fig. 3. Validate that all nonzeros lie within a fixed bandwidth around the diagonal and that nonzero density is low, then persist every staged matrix. The rationale is the paper's finding that the banded pencil arises from four mechanisms acting together: first-order system formulation, T-to-U derivative mapping, Dirichlet preconditioning for boundary rows, and interleaving matrices/coefficients by mode rather than by variable.

## Related tools

- **Python** (implementation language for constructing the Chebyshev basis and staged matrices)
- **numpy** (builds the Chebyshev T_n expansion basis, derivative operators, and the staged banded matrices stored in .npz)
- **scipy** (sparse matrix representation to verify the final pencil is sparse and fully banded) — https://github.com/scipy/scipy
- **Dedalus** (reference framework whose sparse tau-method matrix construction is being reconstructed) — https://github.com/DedalusProject/dedalus
- **matplotlib** (visual inspection of the staged banded matrix structure (cf. Fig. 3 layout))

## Examples

```
import numpy as np; import dedalus.public as d3; M = np.zeros((N, N)); np.savez('poisson_staged_matrices.npz', pre=pre, tau=tau, recombined=recomb, interleaved=inter)
```

## Evaluation signals

- Every nonzero entry of the final assembled pencil lies within a fixed bandwidth around the diagonal (bandedness check)
- Nonzero density of the final pencil matrix is low (sparse)
- Each of the four stages is present and distinct in the .npz output: pre-conversion, tau-augmented, recombined, and interleaved
- Dirichlet conditions u(±1)=0 are exactly satisfied by the retained modes after basis recombination
- Row/column ordering of the final pencil matches the staged mode-interleaved layout of Fig. 3

## Limitations

- The four-mechanism decomposition (first-order formulation, T-to-U conversion, tau rows, basis recombination, mode interleaving) is specific to the paper's Chebyshev Poisson formulation of Eq. (19); other equations or boundary types (e.g. Neumann handled via Dirichlet preconditioning here) may need modified tau-row placement
- Correct bandedness depends on expressing derivative terms in the U test basis; skipping the T-to-U conversion destroys the sparsity structure
- Reconstruction is numpy-based and validated structurally (bandwidth, density) rather than against a Dedalus runtime solve in this card

## Evidence

- [methods] Dedalus is a flexible framework for solving PDEs with spectral methods.: [evidence span withheld — anchor status: not_found]
- [intro] Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification.: "Dedalus is written primarily in Python and features an easy-to-use interface with symbolic vectorial equation specification."
- [readme] Summary: A flexible framework for solving PDEs with modern spectral methods.: [evidence span withheld — anchor status: external_doc]
