---
name: power-spectrum-differentiation
description: Use when gradients of cosmological observables (P(k) or D_l spectra)
  with respect to parameters like (h, Ω_c0, Ω_b0) are needed for Fisher forecasting,
  gradient-based MCMC, emulator training, or sensitivity analyses, and exact (machine-precision)
  derivatives are preferred over finite differences.
when_to_use_negative:
- When only spectrum values (not derivatives) are needed — the differentiation machinery
  and Jacobian compilation add cost without benefit
- When fast reverse-mode AD of a scalar loss function is required — the paper explicitly
  lists this as not yet implemented, so forward-mode Jacobians over the full parameter
  vector are the available path
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0102
tools:
- name: SymBoltz
  role: Symbolic-numeric Einstein–Boltzmann solver whose models are compiled with
    CosmologyProblem (jac = true, sparse = true) and differentiated; spectra computed
    with spectrum_matter and spectrum_cmb
  repo: github.com/hersle/SymBoltz.jl
- name: ForwardDiff.jl
  role: Forward-mode automatic differentiation used to compute exact Jacobians (ForwardDiff.jacobian,
    ForwardDiff.jacobian! with DiffResults) of spectra with respect to parameters
- name: FiniteDiff.jl
  role: Builds central finite-difference Jacobians from CLASS outputs at perturbed
    parameter values for cross-validation
- name: CLASS
  role: Reference Einstein–Boltzmann solver run via CLASSProblem at perturbed parameters
    to supply finite-difference derivative benchmarks
  repo: github.com/lesgourg/class_public
- name: Plots
  role: Renders the derivative-agreement comparison plots between SymBoltz AD Jacobians
    and CLASS finite-difference Jacobians
provenance:
  source_task_ids:
  - task_008
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:3dcdb8d55b0fca85372bef39b90a6e066aef5d92560ef837d51cdc341e83a8e5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CLASS
  - DiffResults.jl
  - FiniteDiff.jl
  - ForwardDiff.jl
  - Julia
  - Plots
  - SymBoltz
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# power-spectrum-differentiation

## Summary

Compute exact derivatives of the matter power spectrum P(k) and CMB spectra D_l (TT/EE/TE) with respect to cosmological parameters by wrapping SymBoltz spectra as a differentiable function of the parameter vector and applying ForwardDiff.jacobian, optionally validated against finite-difference Jacobians from CLASS.

## When to use

Use when gradients of cosmological observables (P(k) or D_l spectra) with respect to parameters like (h, Ω_c0, Ω_b0) are needed for Fisher forecasting, gradient-based MCMC, emulator training, or sensitivity analyses, and exact (machine-precision) derivatives are preferred over finite differences.

## When NOT to use

- When only spectrum values (not derivatives) are needed — the differentiation machinery and Jacobian compilation add cost without benefit
- When fast reverse-mode AD of a scalar loss function is required — the paper explicitly lists this as not yet implemented, so forward-mode Jacobians over the full parameter vector are the available path

## Inputs

- SymBoltz symbolic cosmological model (ΛCDM or w0waCDM) with parameter values (e.g. h = 0.7, Ωc0 = 0.3, Ωb0 = 0.05, YHe = 0.25, Tγ0 = 2.7, Neff = 3.046; or w0 = -0.9, wa = 0.1, c_s^2 = 0.9)
- Log-spaced wavenumber grid ks (e.g. 10^0.5–10^3.5, ~100 points) and multipole range ls (e.g. 25:3000)
- Differentiable scalar or vector function of the parameter vector returning spectrum values (matter P(k) or CMB D_l)

## Outputs

- Jacobian matrix of spectrum values (or logarithmic derivatives, e.g. dlgP_dlgθs) with respect to cosmological parameters
- Values and derivatives computed simultaneously via DiffResults.jacobian! (e.g. Pres)
- Comparison table/plots of SymBoltz ForwardDiff Jacobians vs CLASS central finite-difference Jacobians (relative/absolute differences)

## How to apply

Build the SymBoltz cosmological model (ΛCDM or w0waCDM with lmax = 6, reionization on, Hswitch = 1, Heswitch = 6) and compile it with CosmologyProblem setting jac = true and sparse = true so an analytical sparse Jacobian is generated. Solve the problem on a log-spaced k-grid (e.g. 10^0.5–10^3.5, ~100 points) and compute spectra via spectrum_matter for P(k) or spectrum_cmb for D_l modes [:TT, :EE, :TE] over ls = 25:3000 with SphericalBesselCache(25:25:3000) and normalization = :Dl. Wrap the spectra as a function of the parameter vector (e.g. h, Ω_c0, Ω_b0) in log-space and evaluate the Jacobian with ForwardDiff.jacobian, using DiffResults.jacobian! to obtain values and derivatives simultaneously (e.g. dlgP_dlgθs = ForwardDiff.jacobian(lgP, log10.(θ))). For external validation, run CLASS via CLASSProblem at perturbed parameter values, build central finite-difference Jacobians with FiniteDiff.jl, interpolate onto the common k/ℓ grid with DataInterpolations LinearInterpolation, and compare with plot_compare at tol = 7e-11. Because AD is exact, accept derivatives when they agree with the finite-difference reference within the paper's 0.1% spectra-agreement level with no asserted tolerance check failing.

## Related tools

- **SymBoltz** (Symbolic-numeric Einstein–Boltzmann solver whose models are compiled with CosmologyProblem (jac = true, sparse = true) and differentiated; spectra computed with spectrum_matter and spectrum_cmb) — github.com/hersle/SymBoltz.jl
- **ForwardDiff.jl** (Forward-mode automatic differentiation used to compute exact Jacobians (ForwardDiff.jacobian, ForwardDiff.jacobian! with DiffResults) of spectra with respect to parameters)
- **FiniteDiff.jl** (Builds central finite-difference Jacobians from CLASS outputs at perturbed parameter values for cross-validation)
- **CLASS** (Reference Einstein–Boltzmann solver run via CLASSProblem at perturbed parameters to supply finite-difference derivative benchmarks) — github.com/lesgourg/class_public
- **Plots** (Renders the derivative-agreement comparison plots between SymBoltz AD Jacobians and CLASS finite-difference Jacobians)

## Examples

```
prob = CosmologyProblem(M, p; jac = true, sparse = true); sol = solve(prob, ks); dlgP_dlgθs = ForwardDiff.jacobian(lgP, log10.(θ))
```

## Evaluation signals

- AD Jacobians of D_l(TT)/∂p agree with CLASS central finite-difference Jacobians within the asserted absolute tolerances (plot_compare tol = 7e-11 comparison path)
- Spectra themselves agree with established codes (CLASS) up to 0.1% at standard precision levels
- ForwardDiff gradients match finite-difference references element-wise with relative errors consistent with exactness (near machine precision relative to FD noise)
- Jacobian is compiled as analytical and sparse (jac = true, sparse = true) and solver runs with multithreading across k-modes for performance comparable to approximation-based codes
- DiffResults.jacobian! returns values and derivatives in a single pass that reproduce separately computed spectrum values

## Limitations

- Fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented in SymBoltz (listed as future work); only forward-mode is available, so cost scales with number of differentiated parameters
- Derivatives are exact only up to solver tolerances; comparisons against CLASS inherit the 0.1% spectra-agreement level and finite-difference truncation error
- Run Julia with multiple threads for competitive performance; BLAS backend swaps like MKL.jl (Intel) or AppleAccelerate.jl (Macs) may be needed

## Evidence

- [abstract] Automatic differentiation gives exact derivatives of any output with respect to any input, important for gradient-based MCMC sampling, Fisher forecasting, emulator training, and sensitivity analyses.: "Automatic differentiation gives exact derivatives of any output with respect to any input, which is important for gradient-based Markov chain Monte Carlo (MCMC) sampling in large parameter spaces,"
- [methods] ForwardDiff jacobian workflow: "dlgP_dlgθs = ForwardDiff.jacobian(lgP, log10.(θ))"
- [methods] Analytical sparse Jacobian compilation: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [abstract] Spectra agreement threshold: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [readme] CLASS repository reference: [evidence span withheld — anchor status: external_doc]
