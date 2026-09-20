---
name: cosmological-parameter-sensitivity-benchmarking
description: Use when you need exact derivatives of cosmological observables (power
  spectra P(k), CMB D_l) with respect to parameters like h, Ωc0, Ωb0, YHe for Fisher
  forecasting, gradient-based MCMC, or AD runtime benchmarking.
when_to_use_negative:
- When only a one-off spectrum or model solution is needed without any derivatives
  — the forward/reverse-mode AD machinery and k-grid χ² loss add cost for no benefit
- When the target code is not differentiable-compatible (SymBoltz's advantage is exact
  AD of any output with respect to any input; finite-difference tools like FiniteDiff.jl
  would give only approximate derivatives)
- When comparing against approximation-switching codes is the goal but spectra must
  match only at the ~0.1% level achievable 'with standard precision levels' — tighter
  tolerances may require stricter solver settings
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3360
tools:
- name: SymBoltz
  role: Symbolic-numeric linear Einstein-Boltzmann solver providing the differentiable
    cosmological model, CosmologyProblem compilation with analytical sparse Jacobian,
    and spectrum_matter/spectrum_cmb observables
  repo: https://github.com/hersle/SymBoltz.jl
- name: ForwardDiff.jl
  role: Forward-mode automatic differentiation used to compute reference gradients/Jacobians
    of the scalar loss (ForwardDiff.gradient, ForwardDiff.jacobian, DiffResults.jacobian!)
- name: CLASS
  role: Established Einstein-Boltzmann solver used as external reference to validate
    SymBoltz spectra agreement up to 0.1%
  repo: https://github.com/lesgourg/class_public/
- name: Julia
  role: Implementation language; run with multiple threads since SymBoltz parallelizes
    integration of perturbation k-modes with multithreading
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:768492fce58ec4d2b5c2071a77c9939e957cfda996aea1d6d02e79e3e270c2db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - CLASS
  - ForwardDiff.jl
  - Julia
  - SymBoltz
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cosmological-parameter-sensitivity-benchmarking

## Summary

Benchmark cosmological parameter sensitivity by differentiating scalar loss functions (e.g. weighted χ² over matter/CMB spectra) with respect to cosmological parameters in SymBoltz, comparing forward-mode gradients from ForwardDiff against a reverse-mode AD path, and validating spectra agreement with CLASS to 0.1%.

## When to use

Use when you need exact derivatives of cosmological observables (power spectra P(k), CMB D_l) with respect to parameters like h, Ωc0, Ωb0, YHe for Fisher forecasting, gradient-based MCMC, or AD runtime benchmarking.

## When NOT to use

- When only a one-off spectrum or model solution is needed without any derivatives — the forward/reverse-mode AD machinery and k-grid χ² loss add cost for no benefit
- When the target code is not differentiable-compatible (SymBoltz's advantage is exact AD of any output with respect to any input; finite-difference tools like FiniteDiff.jl would give only approximate derivatives)
- When comparing against approximation-switching codes is the goal but spectra must match only at the ~0.1% level achievable 'with standard precision levels' — tighter tolerances may require stricter solver settings

## Inputs

- ΛCDM SymBoltz model with Planck18-like parameter vector (h, Ωc0, Ωb0, YHe, Tγ0, Neff, mh_eV)
- CosmologyProblem compiled with analytical sparse Jacobian (jac = true, sparse = true)
- Log-spaced k-grid (~10^-1 to 10^4, ~100 steps) for power-spectrum evaluation
- Weighted χ² scalar loss definition over spectrum_matter or spectrum_cmb outputs

## Outputs

- Element-wise gradient comparison of reverse-mode vs ForwardDiff reference (relative differences)
- Wall-clock runtime measurements and runtime ratio of forward- vs reverse-mode gradient paths
- gradient_comparison.csv with correctness (relative gradient error) and runtime columns per differentiated parameter

## How to apply

Build a ΛCDM SymBoltz model and compile a CosmologyProblem with an analytical sparse Jacobian (jac = true, sparse = true) using Planck18-like values (h = 0.7, Ωc0 = 0.3, Ωb0 = 0.05, YHe = 0.25, Tγ0 = 2.7, Neff = 3.046, mh_eV = 0.02). Define a scalar loss as a weighted χ² over P(k) from spectrum_matter (or D_l from spectrum_cmb) on a log-spaced k-grid spanning roughly 10^-1 to 10^4 in ~100 steps. Compute reference gradients of the loss with respect to the parameter vector using ForwardDiff.gradient/ForwardDiff.jacobian, optionally capturing values and derivatives together with DiffResults.jacobian!. Implement the reverse-mode AD path (listed by the paper as missing future work) for the same loss, then compare gradients element-wise (relative differences; AD should give near machine-precision agreement) and measure wall-clock runtime of both paths, recording results in gradient_comparison.csv.

## Related tools

- **SymBoltz** (Symbolic-numeric linear Einstein-Boltzmann solver providing the differentiable cosmological model, CosmologyProblem compilation with analytical sparse Jacobian, and spectrum_matter/spectrum_cmb observables) — https://github.com/hersle/SymBoltz.jl
- **ForwardDiff.jl** (Forward-mode automatic differentiation used to compute reference gradients/Jacobians of the scalar loss (ForwardDiff.gradient, ForwardDiff.jacobian, DiffResults.jacobian!))
- **CLASS** (Established Einstein-Boltzmann solver used as external reference to validate SymBoltz spectra agreement up to 0.1%) — https://github.com/lesgourg/class_public/
- **Julia** (Implementation language; run with multiple threads since SymBoltz parallelizes integration of perturbation k-modes with multithreading)

## Examples

```
prob = CosmologyProblem(M, p; jac = true, sparse = true); Pres = ForwardDiff.jacobian!(Pres, lgP, log10.(θ))
```

## Evaluation signals

- Reverse-mode gradients agree with the ForwardDiff reference to near machine-precision relative error (AD gives exact derivatives)
- Spectra used in the loss agree with established codes (e.g. CLASS) up to 0.1% at standard precision levels
- gradient_comparison.csv contains relative gradient error and runtime columns for all differentiated parameters
- Runtime ratio of reverse-mode vs forward-mode gradient evaluation is reported
- Jacobian sparsity is exploited: CosmologyProblem compiled with jac = true, sparse = true and perturbation k-modes integrated in parallel

## Limitations

- Fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented in SymBoltz — the paper explicitly lists it as needed future work, so the reverse-mode path must be implemented by the practitioner
- More work is needed to implement additional features in SymBoltz generally
- Spectra agreement with established codes holds only up to 0.1% with standard precision levels, so validation tolerances below that are not supported by default settings

## Evidence

- [abstract] More work is needed to implement additional features and for fast reverse-mode automatic differentiation of scalar loss functions.: "More work is needed to implement additional features and for fast reverse-mode automatic differentiation of scalar loss functions."
- [methods] ForwardDiff jacobian of log power spectrum: "dlgP_dlgθs = ForwardDiff.jacobian(lgP, log10.(θ))"
- [methods] Compile problem with analytical sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [abstract] Spectra agree with established codes: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [abstract] AD gives exact derivatives for MCMC/forecasting: [evidence span withheld — anchor status: not_found]
- [readme] README describes SymBoltz.jl purpose: "SymBoltz.jl is a Julia package for solving the linear Einstein-Boltzmann equations in cosmology."
