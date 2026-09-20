---
name: sparse-banded-matrix-verification
description: 'Use when auditing or reconstructing a sparse tau-method linear system:
  you have Chebyshev T_n differential operators and need to confirm that after T-to-U
  conversion, Dirichlet basis recombination, and mode-interleaving, the resulting
  pencil is banded and sparse.'
when_to_use_negative:
- The problem is a dense spectral eigenvalue solve (e.g. the paper's Mathieu problem
  solved with Fourier-basis dense solves) rather than a sparse tau-method system to
  be bandedness-checked.
- The discretization is non-spectral (finite differences/elements) or uses non-polynomial
  bases, so the T-to-U conversion and Dirichlet recombination mechanisms do not apply.
- Boundary conditions are non-Dirichlet, since the recombination/preconditioning rows
  here specifically enforce u(±1)=0.
edam_operation: http://edamontology.org/operation_2426
tools:
- name: numpy
  role: Constructs the Chebyshev T_n basis, differential operators, tau boundary rows,
    and performs the nonzero-pattern/bandwidth/density checks
- name: scipy
  role: Sparse-matrix handling for the assembled banded pencil
  repo: https://github.com/scipy/scipy
- name: Dedalus
  role: Reference implementation of the sparse tau-method matrix construction pipeline
    being reconstructed and verified
  repo: https://github.com/DedalusProject/dedalus
- name: matplotlib
  role: Visualizing staged matrix sparsity patterns (Fig. 3-style layout comparison)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1103/physrevresearch.2.023068
    title: 'Dedalus: A flexible framework for numerical simulations with spectral
      methods'
  source_content_hash: sha256:e7ba7b6b47b2573d3e0415f109182c7ec472a5ba5b1a21fcf3d099b85ff8ba87
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sparse-banded-matrix-verification

## Summary

Reconstructs and audits the staged sparse tau-method matrices for a 1D Chebyshev Poisson problem (Eq. 19), verifying that the assembled pencil is fully banded with low nonzero density, as in Dedalus's matrix construction pipeline (Fig. 3).

## When to use

Use when auditing or reconstructing a sparse tau-method linear system: you have Chebyshev T_n differential operators and need to confirm that after T-to-U conversion, Dirichlet basis recombination, and mode-interleaving, the resulting pencil is banded and sparse.

## When NOT to use

- The problem is a dense spectral eigenvalue solve (e.g. the paper's Mathieu problem solved with Fourier-basis dense solves) rather than a sparse tau-method system to be bandedness-checked.
- The discretization is non-spectral (finite differences/elements) or uses non-polynomial bases, so the T-to-U conversion and Dirichlet recombination mechanisms do not apply.
- Boundary conditions are non-Dirichlet, since the recombination/preconditioning rows here specifically enforce u(±1)=0.

## Inputs

- Chebyshev T_n expansion basis and second-derivative operator for the 1D Poisson problem (Eq. 19) on x in [-1, 1] (numpy arrays / operator matrices)
- T-to-U conversion operator following the paper's ultraspherical conversion relations
- Dirichlet boundary conditions u(±1)=0 for the Poisson problem

## Outputs

- poisson_staged_matrices.npz containing the four staged matrices: pre-conversion, tau-augmented, recombined, and mode-interleaved pencil
- Bandedness summary: maximum bandwidth of nonzero entries around the diagonal and nonzero density of the final assembled pencil

## How to apply

Build the Chebyshev T_n expansion basis and the second-derivative operator for the 1D Poisson problem of Eq. (19) on x in [-1, 1] with numpy, and form the T-to-U test-basis conversion operator so derivative terms land in the U (ultraspherical) basis. Append tau boundary rows enforcing u(±1)=0 in place of the highest-order coefficient rows, then apply Dirichlet basis recombination so retained variable modes exactly satisfy the boundary conditions. Assemble the final pencil with mode-interleaved row/column ordering (equation rows and variable columns alternating by mode index) matching the staged layout of Fig. 3. Validate bandedness by checking that all nonzero entries lie within a fixed bandwidth around the diagonal and that nonzero density is low, then save all four staged matrices (pre-conversion, tau-augmented, recombined, interleaved) plus the bandedness summary to poisson_staged_matrices.npz.

## Related tools

- **numpy** (Constructs the Chebyshev T_n basis, differential operators, tau boundary rows, and performs the nonzero-pattern/bandwidth/density checks)
- **scipy** (Sparse-matrix handling for the assembled banded pencil) — https://github.com/scipy/scipy
- **Dedalus** (Reference implementation of the sparse tau-method matrix construction pipeline being reconstructed and verified) — https://github.com/DedalusProject/dedalus
- **matplotlib** (Visualizing staged matrix sparsity patterns (Fig. 3-style layout comparison))

## Examples

```
import numpy as np
i,j = np.nonzero(A); bw = int(np.abs(i-j).max()); dens = i.size/A.size
np.savez('poisson_staged_matrices.npz', A=A, bandwidth=bw, density=dens)
```

## Evaluation signals

- All nonzeros of the final pencil lie within a fixed bandwidth around the diagonal (max |i-j| over nonzero entries is small and fixed, not O(N)).
- Nonzero density of the assembled pencil is low (sparsity check), consistent with a fully banded structure.
- Boundary rows of the tau-augmented stage enforce u(±1)=0 exactly; recombined-stage modes satisfy the Dirichlet conditions identically.
- The four staged matrices (pre-conversion, tau-augmented, recombined, interleaved) are all present and individually stored in poisson_staged_matrices.npz.
- Interleaved ordering alternates equation rows and variable columns by mode index, matching the staged layout of Fig. 3.

## Limitations

- The pipeline is specific to the four combined mechanisms (first-order system formulation, T-to-U derivative mapping, Dirichlet preconditioning, mode-interleaved assembly); other PDEs or boundary conditions require different recombination rows.
- Bandedness verification confirms sparsity structure only; it does not validate the numerical solution of Eq. (19) itself.

## Evidence

- [methods] bandedness workflow: [evidence span withheld — anchor status: not_found]
- [methods] tau-method mechanisms: [evidence span withheld — anchor status: not_found]
- [results] sparse solve context: [evidence span withheld — anchor status: not_found]
- [readme] framework role: [evidence span withheld — anchor status: external_doc]
