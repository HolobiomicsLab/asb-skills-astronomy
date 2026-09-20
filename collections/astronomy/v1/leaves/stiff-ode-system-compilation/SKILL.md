---
name: stiff-ode-system-compilation
description: Use when a model is specified as symbolic equations and must be solved
  as a stiff implicit ODE system where an analytical sparse Jacobian and automatic
  stage separation (background vs perturbations) are needed for performance and differentiability.
when_to_use_negative:
- The model already exists as hand-written numerical right-hand-side code with a manually
  supplied Jacobian — the symbolic compilation layer offers no benefit there
- Only a scalar quantity of a cheap non-stiff system is needed, where a simple explicit
  solver without Jacobian machinery suffices
- Reverse-mode automatic differentiation of a scalar loss function is required — the
  article states fast reverse-mode AD of scalar loss functions is not yet implemented
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: SymBoltz
  role: 'Provides the symbolic-numeric pipeline: symbolic equation specification,
    automatic stage separation, and symbolic generation of analytical sparse Jacobians
    via CosmologyProblem(M, p; jac = true, sparse = true)'
  repo: https://github.com/hersle/SymBoltz.jl
- name: Julia
  role: Host language; models are symbolic Julia Systems compiled to fast numerical
    functions; multithreading should be enabled for k-mode integration
- name: ForwardDiff.jl
  role: Forward-mode dual-number differentiation used to compute an independent numerical
    reference Jacobian for elementwise verification of the symbolic Jacobian
- name: FiniteDiff.jl
  role: Finite-difference fallback for computing the numerical reference Jacobian
    during verification
- name: Plots
  role: Used to render the sparse Jacobian sparsity-pattern heatmap and the jacobian_verification.png
    agreement plot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:1636069bc5733e19d6ab40ccb745d8b69e93fbcc7801f9745df34dd728eeb195
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
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# stiff-ode-system-compilation

## Summary

Compile a symbolically specified stiff ODE system (e.g. the linear Einstein–Boltzmann equations) into fast numerical functions with an automatically generated analytical, sparse Jacobian, replacing hand-coded Jacobians and numerical sparsity detection.

## When to use

Use when a model is specified as symbolic equations and must be solved as a stiff implicit ODE system where an analytical sparse Jacobian and automatic stage separation (background vs perturbations) are needed for performance and differentiability.

## When NOT to use

- The model already exists as hand-written numerical right-hand-side code with a manually supplied Jacobian — the symbolic compilation layer offers no benefit there
- Only a scalar quantity of a cheap non-stiff system is needed, where a simple explicit solver without Jacobian machinery suffices
- Reverse-mode automatic differentiation of a scalar loss function is required — the article states fast reverse-mode AD of scalar loss functions is not yet implemented

## Inputs

- Symbolic model specification (ModelingToolkit System of equations, e.g. SymBoltz minimal ΛCDM components)
- Named parameter dictionary p with cosmological parameter values
- Set of perturbation wavenumbers k (e.g. ks = [4e0, 4e1, 4e2, 4e3])

## Outputs

- Compiled CosmologyProblem with analytical sparse Jacobian
- Separated sub-systems (background, thermodynamics, perturbations)
- Numerical solution of the stiff ODE system per k-mode
- Jacobian verification report: elementwise deviation, sparsity fraction, jacobian_verification.png

## How to apply

Construct the model as a symbolic System (SymBoltz builds its minimal ΛCDM model — metric, gravity, recombination, baryons, photons, CDM, massless/massive neutrinos, dark energy — with ModelingToolkit), then let the symbolic layer split the single equation system into background, thermodynamics and perturbation sub-systems. Compile the numerical problem with CosmologyProblem(M, p; jac = true, sparse = true) so the Jacobian is generated symbolically and given the analytical sparsity pattern, then solve with high-order implicit solvers over the chosen k-modes (e.g. ks = [4e0, 4e1, 4e2, 4e3]). Verify the compiled Jacobian by comparing it elementwise against a numerical reference computed with ForwardDiff (forward-mode dual numbers) or FiniteDiff, and confirm the nonzero pattern matches exactly (no missing/extra nonzeros); report the sparsity fraction and maximum elementwise deviation. Rationale: SymBoltz's abstract notes that performance comparable to approximation-based codes comes from 'efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods', so trusting the symbolic Jacobian requires exactly this elementwise validation.

## Related tools

- **SymBoltz** (Provides the symbolic-numeric pipeline: symbolic equation specification, automatic stage separation, and symbolic generation of analytical sparse Jacobians via CosmologyProblem(M, p; jac = true, sparse = true)) — https://github.com/hersle/SymBoltz.jl
- **Julia** (Host language; models are symbolic Julia Systems compiled to fast numerical functions; multithreading should be enabled for k-mode integration)
- **ForwardDiff.jl** (Forward-mode dual-number differentiation used to compute an independent numerical reference Jacobian for elementwise verification of the symbolic Jacobian)
- **FiniteDiff.jl** (Finite-difference fallback for computing the numerical reference Jacobian during verification)
- **Plots** (Used to render the sparse Jacobian sparsity-pattern heatmap and the jacobian_verification.png agreement plot)

## Examples

```
prob = CosmologyProblem(M, p; jac = true, sparse = true); sol = solve(prob, ks)
```

## Evaluation signals

- Symbolically generated analytical Jacobian matches the ForwardDiff/FiniteDiff numerical reference elementwise (no missing or extra nonzeros)
- Compiled Jacobian reproduces the analytical sparsity pattern exactly; report the sparsity fraction
- Maximum elementwise deviation between analytical and reference Jacobian is recorded and within solver/AD tolerance
- Splitting into background, thermodynamics and perturbation sub-systems covers all equations of the original single system without loss
- Solutions with the sparse analytical Jacobian agree with established codes (e.g. CLASS spectra within 0.1%) at standard precision levels

## Limitations

- Fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented; forward-mode is the supported route
- The symbolic pipeline is tied to Julia/ModelingToolkit ecosystems (SymBoltz, OrdinaryDiffEq.jl); BLAS backend tuning may require MKL.jl or AppleAccelerate.jl
- Interpretation is limited by missing discussion/changelog content in the indexed document; no extracted limitations, missing artifacts, or reproducibility gaps beyond those stated in the abstract

## Evidence

- [abstract] Symbolic automation of stage separation and Jacobian generation: "Symbolic knowledge enables powerful automation of tasks, such as separating computational stages (e.g., background and perturbations), generating analytical and sparse Jacobian matrices, and"
- [abstract] Performance rationale for analytical sparse Jacobian: "Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods."
- [methods] Concrete compilation call with jac and sparse flags: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [intro] Model structure and automation claims: [evidence span withheld — anchor status: not_found]
- [intro] Verification tooling for Jacobian comparison: "using ForwardDiff, FiniteDiff"
- [abstract] Approximation-free stiff integration claim: "Implicit solvers integrate the full stiff equations at all times without approximations, which greatly simplifies the code."
