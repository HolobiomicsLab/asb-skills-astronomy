---
name: cosmological-parameter-log-derivative-analysis
description: Use when exact derivatives of power spectra or CMB spectra with respect
  to cosmological parameters are needed for Fisher forecasting, gradient-based MCMC,
  emulator training, or sensitivity analyses, and when AD Jacobians must be validated
  against an established Boltzmann solver.
when_to_use_negative:
- When only forward-mode AD of vector-valued outputs is insufficient — the article
  notes fast reverse-mode automatic differentiation of scalar loss functions is not
  yet implemented
- When the comparison target uses approximation switches (tight-coupling, UFA, RSA)
  that SymBoltz deliberately avoids, since approximation-switching artifacts can contaminate
  derivative agreement
- When parameters outside the validated set (h, Ω_c0, Ω_b0) are needed without re-verifying
  agreement, since tolerances were asserted only for these ∂D_l/∂p comparisons
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SymBoltz
  role: symbolic-numeric Einstein–Boltzmann solver whose compiled CosmologyProblem
    provides differentiable spectra
  repo: https://github.com/hersle/SymBoltz.jl
- name: ForwardDiff.jl
  role: forward-mode automatic differentiation to compute exact Jacobians of spectra
    in log-space
- name: FiniteDiff.jl
  role: central finite-difference Jacobians from CLASS runs for cross-validation
- name: CLASS
  role: established Boltzmann solver run via CLASSProblem at perturbed parameters
    to produce reference finite-difference derivatives
  repo: https://github.com/lesgourg/class_public/
- name: DiffResults.jl
  role: evaluates values and derivatives together with ForwardDiff.jacobian!
- name: Plots
  role: renders the derivative-agreement comparison plots
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:a6dc88e1156e3c55e7492f09876f5654934849b3995b1a4735e0c3ec951d8a4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - CLASS
  - DiffResults.jl
  - FiniteDiff.jl
  - ForwardDiff.jl
  - Julia
  - Plots
  - SymBoltz
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cosmological-parameter-log-derivative-analysis

## Summary

Computes exact Jacobians of cosmological observables (matter power spectrum P(k) and CMB D_l spectra) with respect to cosmological parameters (h, Ω_c0, Ω_b0) in log-space using SymBoltz's forward-mode automatic differentiation, and cross-validates them against central finite-difference Jacobians from CLASS.

## When to use

Use when exact derivatives of power spectra or CMB spectra with respect to cosmological parameters are needed for Fisher forecasting, gradient-based MCMC, emulator training, or sensitivity analyses, and when AD Jacobians must be validated against an established Boltzmann solver.

## When NOT to use

- When only forward-mode AD of vector-valued outputs is insufficient — the article notes fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented
- When the comparison target uses approximation switches (tight-coupling, UFA, RSA) that SymBoltz deliberately avoids, since approximation-switching artifacts can contaminate derivative agreement
- When parameters outside the validated set (h, Ω_c0, Ω_b0) are needed without re-verifying agreement, since tolerances were asserted only for these ∂D_l/∂p comparisons

## Inputs

- SymBoltz w0waCDM model with Planck18-like parameters (w0 = -0.9, wa = 0.1, c_s^2 = 0.9, lmax = 6, Hswitch = 1, Heswitch = 6)
- Wavenumber grid ks = 10^0.5–3.5 (100 log-spaced points)
- Multipole grid ls = 25:3000 with SphericalBesselCache(25:25:3000)
- Differentiated parameters (h, Ω_c0, Ω_b0)
- CLASS runs at perturbed parameter values with matching settings (T_cmb, Omega_b, Omega_cdm, w0_fld/wa_fld/cs2_fld, use_ppf = no)

## Outputs

- ForwardDiff.jacobian matrix of log-space spectra (matter P(k) and CMB D_l for :TT, :EE, :TE) with respect to (h, Ω_c0, Ω_b0), including values via DiffResults.jacobian!
- Central finite-difference Jacobian from CLASS (FiniteDiff.jl), interpolated onto the common k/ℓ grid
- Relative/absolute difference tables and derivative-agreement comparison plots (Plots.jl)

## How to apply

Build the SymBoltz w0waCDM model with lmax = 6 (reionization on, Hswitch = 1, Heswitch = 6) and Planck18-like parameters (w0 = -0.9, wa = 0.1, c_s^2 = 0.9), and compile it with CosmologyProblem (jac = true, sparse = true for the analytical+sparse Jacobian). Compute the matter power spectrum with spectrum_matter over ks = 10^0.5–3.5 (100 log-spaced points) and CMB D_l spectra for modes [:TT, :EE, :TE] over ls = 25:3000 with spectrum_cmb using SphericalBesselCache(25:25:3000) and normalization = :Dl. Wrap the spectra as a function of (h, Ω_c0, Ω_b0) in log-space and evaluate Jacobians with ForwardDiff.jacobian, computing values and derivatives simultaneously via DiffResults.jacobian!. For validation, run CLASS through CLASSProblem with matching settings (l_max_g/l_max_pol_g = lmax, newtonian gauge, ad initial conditions, w0_fld/wa_fld/cs2_fld, use_ppf = no), build central finite-difference Jacobians with FiniteDiff.jl, interpolate CLASS derivatives onto the common k/ℓ grid with DataInterpolations LinearInterpolation, and compare with plot_compare at tolerance tol = 7e-11. Derivatives must agree within the paper's reported agreement level (spectra agree with established codes up to 0.1% with standard precision levels).

## Related tools

- **SymBoltz** (symbolic-numeric Einstein–Boltzmann solver whose compiled CosmologyProblem provides differentiable spectra) — https://github.com/hersle/SymBoltz.jl
- **ForwardDiff.jl** (forward-mode automatic differentiation to compute exact Jacobians of spectra in log-space)
- **FiniteDiff.jl** (central finite-difference Jacobians from CLASS runs for cross-validation)
- **CLASS** (established Boltzmann solver run via CLASSProblem at perturbed parameters to produce reference finite-difference derivatives) — https://github.com/lesgourg/class_public/
- **DiffResults.jl** (evaluates values and derivatives together with ForwardDiff.jacobian!)
- **Plots** (renders the derivative-agreement comparison plots)

## Examples

```
Pres = ForwardDiff.jacobian!(Pres, lgP, log10.(θ)) # evaluate values+derivatives
```

## Evaluation signals

- ForwardDiff and finite-difference CLASS Jacobians agree within tol = 7e-11 via plot_compare for ∂D_l/∂p comparisons
- No asserted absolute tolerance check fails for the ∂D_l/∂p comparisons
- Spectra themselves agree with established codes up to 0.1% with standard precision levels
- CLASS derivative values interpolate onto the common k/ℓ grid (DataInterpolations LinearInterpolation) without grid mismatch artifacts
- Jacobian evaluated with values and derivatives computed simultaneously (DiffResults.jacobian!) matches separately evaluated values

## Limitations

- Fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented (more work needed)
- Validation agreement thresholds were asserted for ∂D_l/∂p with respect to (h, Ω_c0, Ω_b0) only; other parameters require their own checks
- CLASS derivatives must be interpolated onto the common k/ℓ grid, introducing interpolation error into the comparison
- Multithreaded k-mode integration requires running Julia with multiple threads for best performance

## Evidence

- [methods] AD Jacobians vs CLASS finite differences with tol=7e-11: [evidence span withheld — anchor status: not_found]
- [methods] Log-space Jacobian via ForwardDiff.jacobian: "dlgP_dlgθs = ForwardDiff.jacobian(lgP, log10.(θ))"
- [methods] Values+derivatives in one pass: "Pres = ForwardDiff.jacobian!(Pres, lgP, log10.(θ)) # evaluate values+derivatives"
- [abstract] 0.1% agreement with established codes: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [abstract] AD enables exact derivatives for MCMC/Fisher: "Automatic differentiation gives exact derivatives of any output with respect to any input, which is important for gradient-based Markov chain Monte Carlo (MCMC) sampling in large parameter spaces,"
- [methods] CLASSProblem w0waCDM comparison setup: "prob = CLASSProblem(
        "write_background" => "yes","
- [intro] SymBoltz is differentiable by design: "**Differentiable**: Get derivatives of any output (e.g. power spectra) with respect to any i"
