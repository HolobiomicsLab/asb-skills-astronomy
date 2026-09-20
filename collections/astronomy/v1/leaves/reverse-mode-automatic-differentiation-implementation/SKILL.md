---
name: reverse-mode-automatic-differentiation-implementation
description: Use when a scalar loss function (e.g. weighted χ² over P(k) on a log-spaced
  k-grid) must be differentiated with respect to many cosmological parameters and
  forward-mode AD is too slow, i.e. gradient-based MCMC, Fisher forecasting or emulator
  training in large parameter spaces.
when_to_use_negative:
- The quantity to differentiate is a vector-valued output (e.g. log power spectra
  themselves) rather than a scalar loss — ForwardDiff.jacobian on the output is already
  supported by SymBoltz
- Only a handful of input parameters are differentiated, so forward-mode AD cost does
  not dominate and reverse mode offers no advantage
- The model was not compiled with an analytical sparse Jacobian (jac = true, sparse
  = true), which the paper identifies as key to differentiable performance
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0102
tools:
- name: SymBoltz
  role: Symbolic-numeric differentiable Einstein–Boltzmann solver providing CosmologyProblem,
    spectrum_matter and spectrum_cmb, whose scalar-loss reverse-mode AD is being implemented
  repo: https://github.com/hersle/SymBoltz.jl
- name: ForwardDiff.jl
  role: Forward-mode AD used to compute exact reference gradients (gradient/jacobian,
    DiffResults.jacobian!) against which the reverse-mode path is validated
- name: Julia
  role: Implementation language; run with multiple threads since SymBoltz parallelizes
    perturbation k-mode integration by default
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:7cbb0999b1801799a9c8c442753e02c5c7e2af44cc13c471a4d37b0d5cf5a695
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - ForwardDiff.jl
  - Julia
  - SymBoltz
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# reverse-mode-automatic-differentiation-implementation

## Summary

Implements and benchmarks fast reverse-mode automatic differentiation (AD) of scalar loss functions (e.g. a weighted χ² over matter power-spectrum values) in SymBoltz, the missing capability the paper lists as future work, using ForwardDiff gradients of a compiled CosmologyProblem as the exact reference.

## When to use

Use when a scalar loss function (e.g. weighted χ² over P(k) on a log-spaced k-grid) must be differentiated with respect to many cosmological parameters and forward-mode AD is too slow, i.e. gradient-based MCMC, Fisher forecasting or emulator training in large parameter spaces.

## When NOT to use

- The quantity to differentiate is a vector-valued output (e.g. log power spectra themselves) rather than a scalar loss — ForwardDiff.jacobian on the output is already supported by SymBoltz
- Only a handful of input parameters are differentiated, so forward-mode AD cost does not dominate and reverse mode offers no advantage
- The model was not compiled with an analytical sparse Jacobian (jac = true, sparse = true), which the paper identifies as key to differentiable performance

## Inputs

- Compiled SymBoltz CosmologyProblem (jac = true, sparse = true) for a ΛCDM model
- Parameter vector θ with Planck18-like values (h, Ωc0, Ωb0, YHe, Tγ0, Neff, mh_eV)
- Log-spaced k-grid (e.g. 10^-1 to 10^4, ~100 steps) and scalar loss definition (weighted χ² over P(k) or D_l)

## Outputs

- Reverse-mode gradients of the scalar loss with respect to all differentiated parameters
- gradient_comparison.csv with relative gradient error (vs ForwardDiff reference) and runtime ratio per parameter
- Wall-clock runtime measurements of forward-mode vs reverse-mode gradient paths

## How to apply

Build the ΛCDM SymBoltz model and compile a CosmologyProblem with an analytical sparse Jacobian (jac = true, sparse = true) using Planck18-like parameters (h = 0.7, Ωc0 = 0.3, Ωb0 = 0.05, YHe = 0.25, Tγ0 = 2.7, Neff = 3.046, mh_eV = 0.02). Define a scalar loss as a weighted χ² over spectrum_matter (or spectrum_cmb) values on a log-spaced k-grid (10^-1 to 10^4 in ~100 steps) and compute reference gradients with ForwardDiff.gradient/ForwardDiff.jacobian, optionally recording values and derivatives together via DiffResults.jacobian!. Implement the reverse-mode AD path for the scalar loss — the feature the paper explicitly lists as missing — and compute the same gradients. Measure wall-clock runtime of both paths and compare gradient values element-wise as relative differences, since AD gives exact derivatives; accept only near machine-precision agreement and report the runtime ratio in gradient_comparison.csv.

## Related tools

- **SymBoltz** (Symbolic-numeric differentiable Einstein–Boltzmann solver providing CosmologyProblem, spectrum_matter and spectrum_cmb, whose scalar-loss reverse-mode AD is being implemented) — https://github.com/hersle/SymBoltz.jl
- **ForwardDiff.jl** (Forward-mode AD used to compute exact reference gradients (gradient/jacobian, DiffResults.jacobian!) against which the reverse-mode path is validated)
- **Julia** (Implementation language; run with multiple threads since SymBoltz parallelizes perturbation k-mode integration by default)

## Examples

```
using SymBoltz, ForwardDiff; prob = CosmologyProblem(M, p; jac=true, sparse=true); g_ref = ForwardDiff.gradient(θ -> chi2_loss(prob, θ), θ0)
```

## Evaluation signals

- Reverse-mode gradients agree with the ForwardDiff reference to near machine-precision relative error (AD gives exact derivatives)
- Spectra entering the loss agree with established codes (e.g. CLASS) up to 0.1% at standard precision levels
- Runtime ratio (reverse-mode vs ForwardDiff wall-clock) is computed and reported for all differentiated parameters
- gradient_comparison.csv contains correctness (relative gradient error) and runtime columns for every differentiated parameter
- Element-wise relative differences between reverse-mode and reference gradients are finite and near zero across the whole parameter vector

## Limitations

- SymBoltz does not yet implement fast reverse-mode AD of scalar loss functions; this is listed as work still needed, so the implementation itself must be supplied
- Reverse-mode speed advantage only materializes for scalar losses differentiated with respect to many parameters; forward-mode remains the reference for per-output derivatives
- Multithreaded k-mode integration means wall-clock runtime comparisons may vary with thread count and machine BLAS backend

## Evidence

- [abstract] Missing feature: fast reverse-mode AD of scalar loss functions: "More work is needed to implement additional features and for fast reverse-mode automatic differentiation of scalar loss functions."
- [abstract] AD gives exact derivatives, motivating gradient-based inference: "Automatic differentiation gives exact derivatives of any output with respect to any input, which is important for gradient-based Markov chain Monte Carlo (MCMC) sampling in large parameter spaces,"
- [methods] Compile the problem with analytical sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [methods] ForwardDiff reference gradients and values+derivatives: "Pres = ForwardDiff.jacobian!(Pres, lgP, log10.(θ)) # evaluate values+derivatives"
- [methods] Matter power spectrum computation used in the loss: "Ps = spectrum_matter(modes, prob, ks)"
- [abstract] Spectra validation threshold against established codes: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [readme] README confirms SymBoltz is a differentiable Julia Einstein–Boltzmann solver: "SymBoltz.jl is a Julia package for solving the linear Einstein-Boltzmann equations in cosmology."
