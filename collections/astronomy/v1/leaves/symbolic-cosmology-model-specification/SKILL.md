---
name: symbolic-cosmology-model-specification
description: Use when building or adapting an Einstein-Boltzmann solver from symbolic
  equations and you need automated stage separation plus analytical sparse Jacobians
  instead of approximation-switching schemes and finite-difference Jacobians.
when_to_use_negative:
- You need fast reverse-mode automatic differentiation of scalar loss functions —
  the paper states this is not yet implemented and more work is needed.
- You want an approximation-switching solver (tight-coupling, ultrarelativistic fluid
  or radiation-streaming approximations); SymBoltz intentionally solves the full equations
  at all times, so this skill is the wrong fit for replicating TCA/UFA/RSA-style codes.
- The model is a hand-coded numerical pipeline with no symbolic equation representation
  — stage separation and symbolic Jacobian generation require the symbolic System
  input.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: SymBoltz
  role: Symbolic-numeric package that specifies the model equations, automates stage
    separation, and generates analytical sparse Jacobians
  repo: https://github.com/hersle/SymBoltz.jl
- name: Julia
  role: Language runtime in which SymBoltz is implemented and the symbolic-numeric
    compilation is executed (run with multiple threads for parallel k-mode integration)
  repo: https://github.com/hersle/SymBoltz.jl
- name: ForwardDiff.jl
  role: Forward-mode dual-number differentiation used to compute the numerical reference
    Jacobian for elementwise validation
- name: FiniteDiff.jl
  role: Finite-difference fallback for computing the numerical reference Jacobian
- name: Plots
  role: Plotting the sparse Jacobian nonzero heatmap and the jacobian_verification.png
    elementwise-agreement figure
- name: CLASS
  role: Established Einstein-Boltzmann solver (via CLASSProblem) used as external
    reference for output agreement
  repo: https://github.com/lesgourg/class_public
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:ded3d8533eff80c4dc7363e23edc80706d50cb3c83fe0d95b4b5d546e70c7f80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - CLASS
  - FiniteDiff.jl
  - ForwardDiff.jl
  - Julia
  - Plots
  - SymBoltz
  - SymBoltz.jl
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# symbolic-cosmology-model-specification

## Summary

Specify cosmological models as symbolic equation systems (via ModelingToolkit in SymBoltz.jl) so the symbolic-numeric compiler can automatically separate computational stages (background, thermodynamics, perturbations) and generate analytical, sparse Jacobians. This replaces hand-coded stage splitting and numerical Jacobians with exact, sparsity-aware generated code.

## When to use

Use when building or adapting an Einstein-Boltzmann solver from symbolic equations and you need automated stage separation plus analytical sparse Jacobians instead of approximation-switching schemes and finite-difference Jacobians.

## When NOT to use

- You need fast reverse-mode automatic differentiation of scalar loss functions — the paper states this is not yet implemented and more work is needed.
- You want an approximation-switching solver (tight-coupling, ultrarelativistic fluid or radiation-streaming approximations); SymBoltz intentionally solves the full equations at all times, so this skill is the wrong fit for replicating TCA/UFA/RSA-style codes.
- The model is a hand-coded numerical pipeline with no symbolic equation representation — stage separation and symbolic Jacobian generation require the symbolic System input.

## Inputs

- Symbolic ΛCDM model specification (equations for metric, gravity, recombination, baryons, photons, cold dark matter, massless and massive neutrinos, dark energy) as a ModelingToolkit System in SymBoltz.jl
- Cosmological parameter values `p` for the model variables
- Set of perturbation wavenumbers ks (e.g. [4e0, 4e1, 4e2, 4e3])

## Outputs

- Background, thermodynamics and perturbation sub-systems split from the single symbolic equation system
- Compiled numerical ODE problem with an analytical, sparse Jacobian (CosmologyProblem)
- Sparse Jacobian sparsity report: number of equations, sparsity fraction, heatmap of nonzeros
- jacobian_verification.png comparing analytical vs. numerical reference Jacobian elementwise

## How to apply

Load a minimal ΛCDM symbolic model specification — equations for metric, gravity, recombination, baryons, photons, cold dark matter, massless and massive neutrinos, and dark energy — and construct it as a symbolic System with ModelingToolkit in SymBoltz.jl. Exploit the symbolic representation to split the single equation system into background, thermodynamics and perturbation sub-systems rather than coding these stages by hand. Compile the numerical problem with `CosmologyProblem(M, p; jac = true, sparse = true)` so the analytical Jacobian is generated symbolically with its analytical sparsity pattern, and inspect the sparse Jacobian (e.g. a heatmap of nonzeros) recording the number of equations and sparsity fraction. To validate, compute a numerical reference Jacobian of the perturbation ODE right-hand side with forward-mode dual numbers (ForwardDiff.jl) or finite differences (FiniteDiff.jl), compare elementwise, and require the analytical Jacobian to reproduce the exact sparsity pattern with no missing or extra nonzeros. The rationale is that implicit solvers integrate the full stiff equations without approximations, so an 'optimal handling of the Jacobian' and sparse matrix methods are what keep performance comparable to approximation-based codes like CLASS.

## Related tools

- **SymBoltz** (Symbolic-numeric package that specifies the model equations, automates stage separation, and generates analytical sparse Jacobians) — https://github.com/hersle/SymBoltz.jl
- **Julia** (Language runtime in which SymBoltz is implemented and the symbolic-numeric compilation is executed (run with multiple threads for parallel k-mode integration)) — https://github.com/hersle/SymBoltz.jl
- **ForwardDiff.jl** (Forward-mode dual-number differentiation used to compute the numerical reference Jacobian for elementwise validation)
- **FiniteDiff.jl** (Finite-difference fallback for computing the numerical reference Jacobian)
- **Plots** (Plotting the sparse Jacobian nonzero heatmap and the jacobian_verification.png elementwise-agreement figure)
- **CLASS** (Established Einstein-Boltzmann solver (via CLASSProblem) used as external reference for output agreement) — https://github.com/lesgourg/class_public

## Examples

```
prob = CosmologyProblem(M, p; jac = true, sparse = true); sol = solve(prob, ks)
```

## Evaluation signals

- The symbolically generated analytical Jacobian matches the ForwardDiff/FiniteDiff reference Jacobian elementwise; report the maximum elementwise deviation.
- The generated Jacobian reproduces the analytical sparsity pattern exactly — no missing or extra nonzeros in the sparse structure.
- Number of equations and sparsity fraction of the Jacobian are recorded from the prototype heatmap of nonzeros.
- Compiled problem is built with `jac = true, sparse = true` in CosmologyProblem; the solver integrates the full stiff equations at all times without approximation switching (TCA/UFA/RSA).
- Output spectra agree with CLASS up to 0.1% at standard precision levels, confirming the symbolic pipeline does not degrade physics accuracy.

## Limitations

- Fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented ('More work is needed').
- Additional features beyond the minimal ΛCDM model may need to be implemented or adapted from the src/models/ library.
- Performance parity with approximation-based codes depends on running Julia with multiple threads and, on some platforms, optimized BLAS backends such as MKL.jl (Intel) or AppleAccelerate.jl (Macs).

## Evidence

- [abstract] Symbolic knowledge enables powerful automation of tasks, such as separating computational stages (e.g., background and perturbations), generating analytical and sparse Jacobian matrices: "Symbolic knowledge enables powerful automation of tasks, such as separating computational stages (e.g., background and perturbations), generating analytical and sparse Jacobian matrices"
- [abstract] Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods.: "Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods."
- [methods] Compile the numerical problem with the analytical sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [intro] ForwardDiff and FiniteDiff are used for numerical reference differentiation: "using ForwardDiff, FiniteDiff"
- [abstract] Implicit solvers integrate the full stiff equations at all times without approximations, which greatly simplifies the code.: "Implicit solvers integrate the full stiff equations at all times without approximations, which greatly simplifies the code."
- [methods] SymBoltz provides a minimal ΛCDM model in addition to component-based models: "In addition to [component-based models](@ref "Cosmologies (full models)"), SymBoltz provides a minimal ΛCDM model"
- [abstract] Output spectra agree with established codes up to 0.1% with standard precision levels.: "Output spectra agree with established codes up to 0.1% with standard precision levels."
