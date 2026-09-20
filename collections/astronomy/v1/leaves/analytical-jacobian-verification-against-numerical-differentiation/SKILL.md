---
name: analytical-jacobian-verification-against-numerical-differentiation
description: Use when an ODE/perturbation solver provides a symbolically or analytically
  generated Jacobian and you must prove it is correct before relying on it for implicit
  integration or AD-based inference.
when_to_use_negative:
- The Jacobian is already differentiated and validated numerically (e.g. via ForwardDiff.jacobian
  of outputs) and no symbolic Jacobian needs verification
- The model contains no symbolic representation, so no analytical Jacobian or sparsity
  pattern exists to verify
- The goal is derivative accuracy of scalar outputs (e.g. logarithmic derivatives
  of P(k) via ForwardDiff.jacobian) rather than verification of an ODE-system Jacobian
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3316
tools:
- name: SymBoltz
  role: Symbolic-numeric package that automatically generates the analytical sparse
    Jacobian and sparsity pattern from the symbolic model equations
  repo: https://github.com/hersle/SymBoltz.jl
- name: ForwardDiff.jl
  role: Forward-mode dual-number automatic differentiation used to compute the independent
    numerical reference Jacobian of the perturbation ODE right-hand side
- name: FiniteDiff.jl
  role: Finite-difference fallback for computing the numerical reference Jacobian
- name: Plots
  role: Plotting the sparse Jacobian nonzeros heatmap and the elementwise agreement
    comparison (jacobian_verification.png)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:47cafe158d14f703716193f3bafe6cbbadefc307ec4da420fc310e4afbabd824
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - FiniteDiff.jl
  - ForwardDiff.jl
  - Julia
  - Plots
  - SymBoltz
  - SymBoltz.jl
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# analytical-jacobian-verification-against-numerical-differentiation

## Summary

Verifies a symbolically generated analytical (sparse) Jacobian — e.g. from SymBoltz's symbolic-numeric pipeline with CosmologyProblem(M, p; jac = true, sparse = true) — against an independently computed numerical Jacobian (ForwardDiff/FiniteDiff), elementwise, and validates the analytical sparsity pattern.

## When to use

Use when an ODE/perturbation solver provides a symbolically or analytically generated Jacobian and you must prove it is correct before relying on it for implicit integration or AD-based inference.

## When NOT to use

- The Jacobian is already differentiated and validated numerically (e.g. via ForwardDiff.jacobian of outputs) and no symbolic Jacobian needs verification
- The model contains no symbolic representation, so no analytical Jacobian or sparsity pattern exists to verify
- The goal is derivative accuracy of scalar outputs (e.g. logarithmic derivatives of P(k) via ForwardDiff.jacobian) rather than verification of an ODE-system Jacobian

## Inputs

- Symbolic cosmological model specification (SymBoltz System built with ModelingToolkit, e.g. minimal ΛCDM)
- Numerical problem compiled with analytical sparse Jacobian: CosmologyProblem(M, p; jac = true, sparse = true)
- Perturbation ODE right-hand side for numerical differentiation

## Outputs

- Analytical sparse Jacobian with its sparsity pattern (e.g. nonzeros heatmap) plus equation count and sparsity fraction
- Numerical reference Jacobian (ForwardDiff dual numbers and/or FiniteDiff finite differences)
- Elementwise comparison plot, e.g. jacobian_verification.png, with maximum elementwise deviation

## How to apply

Construct the symbolic model as a SymBoltz System (e.g. the minimal ΛCDM model covering metric, gravity, recombination, baryons, photons, CDM, massless/massive neutrinos and dark energy) and use its symbolic automation to split the equations into background, thermodynamics and perturbation sub-systems. Compile the numerical problem with CosmologyProblem(M, p; jac = true, sparse = true) so the analytical Jacobian is generated symbolically together with its analytical sparsity pattern, then inspect the sparsity (e.g. a heatmap of nonzeros) and record the number of equations and sparsity fraction. Independently compute a numerical reference Jacobian of the perturbation ODE right-hand side with forward-mode dual numbers (ForwardDiff) and/or finite differences (FiniteDiff). Compare the two Jacobians elementwise (e.g. plotting agreement as jacobian_verification.png) and require both elementwise agreement and exact reproduction of the sparsity pattern — no missing or extra nonzeros — reporting the sparsity fraction and the maximum elementwise deviation.

## Related tools

- **SymBoltz** (Symbolic-numeric package that automatically generates the analytical sparse Jacobian and sparsity pattern from the symbolic model equations) — https://github.com/hersle/SymBoltz.jl
- **ForwardDiff.jl** (Forward-mode dual-number automatic differentiation used to compute the independent numerical reference Jacobian of the perturbation ODE right-hand side)
- **FiniteDiff.jl** (Finite-difference fallback for computing the numerical reference Jacobian)
- **Plots** (Plotting the sparse Jacobian nonzeros heatmap and the elementwise agreement comparison (jacobian_verification.png))

## Examples

```
prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian
```

## Evaluation signals

- Analytical sparse Jacobian matches the numerically differentiated reference Jacobian elementwise (maximum elementwise deviation reported and small)
- Sparsity pattern of the analytical Jacobian exactly reproduces the analytical pattern — no missing and no extra nonzeros
- Number of equations and sparsity fraction are recorded for the compiled perturbation system
- Verification artifacts exist: nonzeros heatmap and jacobian_verification.png comparison plot

## Limitations

- Verification relies on accurate numerical differentiation; ForwardDiff dual numbers or FiniteDiff tolerances must be chosen carefully for the stiff Einstein–Boltzmann system
- SymBoltz's fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented, so derivative workflows beyond forward-mode may be limited
- Validation only confirms correctness of the Jacobian for the compiled sub-systems (background/thermodynamics/perturbations) of the specified model

## Evidence

- [abstract] Symbolic knowledge enables powerful automation of tasks, such as separating computational stages (e.g., background and perturbations), generating analytical and sparse Jacobian matrices, and: "Symbolic knowledge enables powerful automation of tasks, such as separating computational stages (e.g., background and perturbations), generating analytical and sparse Jacobian matrices"
- [methods] prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [abstract] Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods.: "Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods."
- [intro] using ForwardDiff, FiniteDiff: "using ForwardDiff, FiniteDiff"
