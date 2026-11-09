---
name: computational-stage-separation
description: Use when a cosmological model is specified as one symbolic equation system
  (metric, gravity, recombination, baryons, photons, CDM, neutrinos, dark energy)
  and must be decomposed into computational stages with analytical sparse Jacobians
  for implicit stiff ODE integration.
when_to_use_negative:
- The model is already expressed as hand-separated background/perturbation code (e.g.
  a traditional approximation-switching solver like CLASS) — the symbolic automation
  adds no value there
- A dense Jacobian or non-stiff explicit solver is desired; this skill targets implicit
  solvers that benefit from the analytical sparse sparsity pattern
- Reverse-mode automatic differentiation of a scalar loss function is required — the
  paper notes this is not yet implemented
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SymBoltz
  role: Julia package providing the symbolic-numeric pipeline that automates stage
    separation and analytical sparse Jacobian generation
  repo: https://github.com/hersle/SymBoltz.jl
- name: SymBoltz.jl
  role: Package whose symbolic System specification (with ModelingToolkit) is split
    into background/thermodynamics/perturbation sub-systems
  repo: https://github.com/hersle/SymBoltz.jl
- name: ForwardDiff.jl
  role: Forward-mode dual-number differentiation used to compute the numerical reference
    Jacobian for validation
- name: FiniteDiff.jl
  role: Finite-difference reference Jacobian computation for cross-checking the analytical
    Jacobian
- name: Plots
  role: Plotting the sparse Jacobian nonzero heatmap and the jacobian_verification.png
    elementwise-agreement figure
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:8d1ef9abf2997eec4cd06153d8e27b7263b18c7df81c8bee9190606e7aaf9985
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# computational-stage-separation

## Summary

Exploits SymBoltz's symbolic model representation (built with ModelingToolkit) to automatically split a single coupled Einstein–Boltzmann equation system into background, thermodynamics and perturbation sub-systems, and to compile them with an analytically generated, sparse Jacobian via CosmologyProblem(M, p; jac = true, sparse = true).

## When to use

Use when a cosmological model is specified as one symbolic equation system (metric, gravity, recombination, baryons, photons, CDM, neutrinos, dark energy) and must be decomposed into computational stages with analytical sparse Jacobians for implicit stiff ODE integration.

## When NOT to use

- The model is already expressed as hand-separated background/perturbation code (e.g. a traditional approximation-switching solver like CLASS) — the symbolic automation adds no value there
- A dense Jacobian or non-stiff explicit solver is desired; this skill targets implicit solvers that benefit from the analytical sparse sparsity pattern
- Reverse-mode automatic differentiation of a scalar loss function is required — the paper notes this is not yet implemented

## Inputs

- Symbolic ΛCDM model specification (equations for metric, gravity, recombination, baryons, photons, cold dark matter, massless and massive neutrinos, dark energy) as a ModelingToolkit System
- Parameter set p with cosmological parameter values

## Outputs

- Separated background, thermodynamics and perturbation sub-systems
- Compiled numerical problem (CosmologyProblem) with analytical sparse Jacobian and analytical sparsity pattern
- Sparse Jacobian prototype visualization (heatmap of nonzeros) and recorded sparsity fraction
- jacobian_verification.png showing elementwise agreement between analytical and numerical reference Jacobians

## How to apply

Construct the minimal ΛCDM model as a symbolic System with ModelingToolkit in SymBoltz, then rely on the symbolic automation to split the single equation system into background, thermodynamics and perturbation sub-systems rather than hand-coding each stage. Compile the numerical problem with CosmologyProblem(M, p; jac = true, sparse = true) so the analytical Jacobian is generated symbolically and given the analytical sparsity pattern — this 'optimal handling of the Jacobian, and sparse matrix methods' is what keeps the approximation-free implicit integration competitive with approximation-switching codes like CLASS. Inspect the sparse Jacobian prototype (e.g. a heatmap of nonzeros) and record the number of equations and the sparsity fraction. Validate by computing a numerical reference Jacobian of the perturbation ODE right-hand side with forward-mode dual numbers (ForwardDiff) and/or finite differences (FiniteDiff), comparing elementwise and checking that no nonzeros are missing or extra; plot agreement as jacobian_verification.png.

## Related tools

- **SymBoltz** (Julia package providing the symbolic-numeric pipeline that automates stage separation and analytical sparse Jacobian generation) — https://github.com/hersle/SymBoltz.jl
- **SymBoltz.jl** (Package whose symbolic System specification (with ModelingToolkit) is split into background/thermodynamics/perturbation sub-systems) — https://github.com/hersle/SymBoltz.jl
- **ForwardDiff.jl** (Forward-mode dual-number differentiation used to compute the numerical reference Jacobian for validation)
- **FiniteDiff.jl** (Finite-difference reference Jacobian computation for cross-checking the analytical Jacobian)
- **Plots** (Plotting the sparse Jacobian nonzero heatmap and the jacobian_verification.png elementwise-agreement figure)

## Examples

```
prob = CosmologyProblem(M, p; jac = true, sparse = true)  # analytical+sparse Jacobian
```

## Evaluation signals

- The analytical sparse Jacobian matches the numerically differentiated reference Jacobian (ForwardDiff dual numbers and/or FiniteDiff) elementwise, with maximum elementwise deviation reported
- The Jacobian reproduces the analytical sparsity pattern exactly — no missing or extra nonzeros
- The sparsity fraction and total equation count of the compiled system are recorded and consistent with the symbolic model
- Separation yields the expected three sub-systems: background, thermodynamics and perturbations

## Limitations

- Fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented; more work is needed for that feature
- Performance competitiveness depends on Julia being run with multiple threads (SymBoltz parallelizes k-mode integration with multithreading by default) and on optimal BLAS backends such as MKL.jl or AppleAccelerate.jl

## Evidence

- [abstract] Symbolic knowledge enables powerful automation of tasks, such as separating computational stages (e.g., background and perturbations), generating analytical and sparse Jacobian matrices: "Symbolic knowledge enables powerful automation of tasks, such as separating computational stages (e.g., background and perturbations), generating analytical and sparse Jacobian matrices"
- [methods] prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [abstract] Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods.: "Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods."
- [intro] using ForwardDiff, FiniteDiff: "using ForwardDiff, FiniteDiff"
