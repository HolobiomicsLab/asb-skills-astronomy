---
name: sparse-jacobian-sparsity-analysis
description: Use when a large stiff ODE system (e.g. cosmological background+thermodynamics+perturbation
  equations) is compiled from symbolic equations and you need an analytical sparse
  Jacobian that is verified to match the true sparsity pattern and numerical derivatives.
when_to_use_negative:
- The system is small and dense, so symbolic sparsity analysis and sparse linear algebra
  add no benefit over a dense Jacobian
- The model is not specified symbolically and no analytical Jacobian can be generated;
  a purely numerical finite-difference Jacobian must be used instead
- You need reverse-mode (adjoint) derivatives of scalar loss functions — the paper
  notes fast reverse-mode AD of scalar losses is not yet implemented in SymBoltz
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SymBoltz
  role: Symbolic-numeric package that separates computational stages (background/perturbations)
    and generates the analytical sparse Jacobian for the Einstein–Boltzmann system
  repo: github.com/hersle/SymBoltz.jl
- name: ForwardDiff.jl
  role: Forward-mode dual-number differentiation to compute the numerical reference
    Jacobian of the perturbation ODE right-hand side
  repo: github.com/hersle/SymBoltz.jl
- name: FiniteDiff.jl
  role: Finite-difference alternative for computing the numerical reference Jacobian
    used in verification
  repo: github.com/hersle/SymBoltz.jl
- name: Plots
  role: Visualizing the sparse Jacobian sparsity pattern (heatmap of nonzeros) and
    the analytical-vs-numerical Jacobian agreement plot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:02aeedf7af2fe76f69528771af16fe0114c06797d54f9e51440b4ff3a2143f55
schema_version: 0.2.0
metadata:
  license_tier: open
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
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Sparse Jacobian sparsity analysis

## Summary

A method for symbolically generating an analytical, sparse Jacobian matrix for a stiff ODE system (here the linear Einstein–Boltzmann equations compiled with SymBoltz/ModelingToolkit), then validating it elementwise against a numerically differentiated reference Jacobian.

## When to use

Use when a large stiff ODE system (e.g. cosmological background+thermodynamics+perturbation equations) is compiled from symbolic equations and you need an analytical sparse Jacobian that is verified to match the true sparsity pattern and numerical derivatives.

## When NOT to use

- The system is small and dense, so symbolic sparsity analysis and sparse linear algebra add no benefit over a dense Jacobian
- The model is not specified symbolically and no analytical Jacobian can be generated; a purely numerical finite-difference Jacobian must be used instead
- You need reverse-mode (adjoint) derivatives of scalar loss functions — the paper notes fast reverse-mode AD of scalar losses is not yet implemented in SymBoltz

## Inputs

- Symbolic model specification (SymBoltz System built with ModelingToolkit; equations for metric, gravity, recombination, baryons, photons, CDM, massless and massive neutrinos, dark energy)
- Parameter values p for the cosmological model
- Numerical Jacobian reference computed with ForwardDiff (forward-mode duals) and/or FiniteDiff

## Outputs

- Compiled numerical problem with symbolic analytical sparse Jacobian (CosmologyProblem with jac = true, sparse = true)
- Sparsity pattern prototype (e.g. heatmap of nonzeros) with equation count and sparsity fraction
- Elementwise agreement plot of analytical vs numerical Jacobian (e.g. jacobian_verification.png), with maximum elementwise deviation

## How to apply

Construct the model as a symbolic System (in SymBoltz, the minimal ΛCDM model spanning metric, gravity, recombination, baryons, photons, cold dark matter, massless and massive neutrinos, and dark energy) and let the symbolic automation split it into background, thermodynamics and perturbation sub-systems. Compile the numerical problem with CosmologyProblem(M, p; jac = true, sparse = true) so the analytical Jacobian is generated symbolically and given the analytical sparsity pattern. Inspect the sparse Jacobian prototype (e.g. a heatmap of nonzeros) and record the number of equations and the sparsity fraction. Build an independent reference Jacobian of the perturbation ODE right-hand side with forward-mode dual numbers (ForwardDiff) and/or finite differences (FiniteDiff). Validate by comparing the analytical and reference Jacobians elementwise and confirming the sparsity pattern matches exactly (no missing or extra nonzeros); report the sparsity fraction and maximum elementwise deviation. The rationale is that 'optimal handling of the Jacobian' and sparse matrix methods are what keep implicit solvers of the full stiff equations competitive with approximation-based codes.

## Related tools

- **SymBoltz** (Symbolic-numeric package that separates computational stages (background/perturbations) and generates the analytical sparse Jacobian for the Einstein–Boltzmann system) — github.com/hersle/SymBoltz.jl
- **ForwardDiff.jl** (Forward-mode dual-number differentiation to compute the numerical reference Jacobian of the perturbation ODE right-hand side) — github.com/hersle/SymBoltz.jl
- **FiniteDiff.jl** (Finite-difference alternative for computing the numerical reference Jacobian used in verification) — github.com/hersle/SymBoltz.jl
- **Plots** (Visualizing the sparse Jacobian sparsity pattern (heatmap of nonzeros) and the analytical-vs-numerical Jacobian agreement plot)

## Examples

```
M = SymBoltz.ΛCDM(); p = ...; prob = CosmologyProblem(M, p; jac = true, sparse = true)
```

## Evaluation signals

- Analytical sparse Jacobian matches the ForwardDiff/FiniteDiff reference Jacobian elementwise with a small maximum elementwise deviation
- Symbolically generated sparsity pattern exactly reproduces the analytical pattern: no missing and no extra nonzeros
- Sparsity fraction and number of equations are reported and consistent with the heatmap of nonzeros
- Compiled problem uses jac = true, sparse = true, confirming the analytical+sparse Jacobian path was taken
- Output spectra agree with established codes (e.g. CLASS) up to 0.1%, indirectly confirming the Jacobian used by the implicit solver is correct

## Limitations

- Fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented, limiting adjoint-based gradient use
- The sparse-Jacobian workflow is tied to symbolically specified models (ModelingToolkit Systems); it cannot be applied to hand-coded right-hand sides without a symbolic layer
- Forward-mode AD of the Jacobian scales with parameter-space size; the paper notes more work is needed for additional features and fast reverse-mode differentiation

## Evidence

- [abstract] Symbolic knowledge enables powerful automation of tasks, such as separating computational stages (e.g., background and perturbations), generating analytical and sparse Jacobian matrices: "Symbolic knowledge enables powerful automation of tasks, such as separating computational stages (e.g., background and perturbations), generating analytical and sparse Jacobian matrices"
- [abstract] Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods.: "Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods."
- [methods] prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [intro] using ForwardDiff, FiniteDiff: "using ForwardDiff, FiniteDiff"
- [abstract] Implicit solvers integrate the full stiff equations at all times without approximations: "Implicit solvers integrate the full stiff equations at all times without approximations, which greatly simplifies the code."
