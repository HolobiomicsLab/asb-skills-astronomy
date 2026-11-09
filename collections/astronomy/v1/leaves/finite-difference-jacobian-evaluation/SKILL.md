---
name: finite-difference-jacobian-evaluation
description: 'Use when validating automatic-differentiation derivatives of cosmological
  observables against an independent solver: you need a reference finite-difference
  Jacobian of D_l(TT/EE/TE) or P(k) w.r.t. (h, Ω_c0, Ω_b0) computed with CLASS.'
when_to_use_negative:
- When the derivative target is a SymBoltz output that ForwardDiff can differentiate
  directly — AD gives exact derivatives and finite differences are only needed as
  an independent cross-check against CLASS
- When comparing against codes other than CLASS or parameters outside the varied set
  (h, Ω_c0, Ω_b0), since the comparison settings (gauge, initial conditions, PPF flag,
  w0waCDM fluid parameters) are tuned to match the w0waCDM model configuration
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: FiniteDiff.jl
  role: computes the central finite-difference Jacobians from CLASS outputs at perturbed
    parameter values
  repo: github.com/hersle/SymBoltz.jl
- name: CLASS
  role: reference Einstein-Boltzmann solver evaluated at perturbed parameter values
    via CLASSProblem to produce finite-difference derivative targets
  repo: github.com/lesgourg/class_public
- name: ForwardDiff.jl
  role: provides the automatic-differentiation Jacobians being validated against the
    finite-difference reference
  repo: github.com/hersle/SymBoltz.jl
- name: DiffResults.jl
  role: evaluates values and derivatives together via DiffResults.jacobian!
- name: DataInterpolations
  role: LinearInterpolation of CLASS derivative values onto the common k/ℓ grid before
    comparison
- name: Plots
  role: plot_compare visualization of the derivative-agreement comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:ed74cd1c2c073ad0357e4c3186567eddd33d328db7f54cf6cfaf82bec754b00d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CLASS
  - DataInterpolations
  - DiffResults.jl
  - FiniteDiff.jl
  - ForwardDiff.jl
  - Julia
  - Plots
  - SymBoltz
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# finite-difference-jacobian-evaluation

## Summary

Build central finite-difference Jacobians of cosmological spectra (matter power spectrum P(k), CMB D_l for TT/EE/TE) with respect to parameters (h, Ω_c, Ω_b) by evaluating CLASS at perturbed parameter values via CLASSProblem, then compare against SymBoltz's ForwardDiff.jacobian automatic-differentiation Jacobians on a common interpolated grid.

## When to use

Use when validating automatic-differentiation derivatives of cosmological observables against an independent solver: you need a reference finite-difference Jacobian of D_l(TT/EE/TE) or P(k) w.r.t. (h, Ω_c0, Ω_b0) computed with CLASS.

## When NOT to use

- When the derivative target is a SymBoltz output that ForwardDiff can differentiate directly — AD gives exact derivatives and finite differences are only needed as an independent cross-check against CLASS
- When comparing against codes other than CLASS or parameters outside the varied set (h, Ω_c0, Ω_b0), since the comparison settings (gauge, initial conditions, PPF flag, w0waCDM fluid parameters) are tuned to match the w0waCDM model configuration

## Inputs

- Cosmological parameter vector θ = (h, Ω_c0, Ω_b0) at Planck18-like values (w0 = -0.9, wa = 0.1, c_s^2 = 0.9)
- wavenumber grid ks = 10^0.5–3.5 (100 log-spaced points) for the matter power spectrum
- multipole range ls = 25:3000 with SphericalBesselCache(25:25:3000) for CMB spectra modes [:TT, :EE, :TE]
- CLASS configuration (CLASSProblem settings: l_max_g/l_max_pol_g, newtonian gauge, ad initial conditions, w0_fld/wa_fld/cs2_fld, use_ppf = no)

## Outputs

- Central finite-difference Jacobian matrix ∂spectra/∂(h, Ω_c0, Ω_b0) from FiniteDiff.jl
- Interpolated CLASS derivative values on the common k/ℓ grid (DataInterpolations LinearInterpolation)
- Relative/absolute difference table and plot_compare figure against the ForwardDiff.jacobian AD derivatives

## How to apply

Wrap the spectra as a function of (h, Ω_c0, Ω_b0) in log-space, so finite-difference steps probe relative parameter changes. For each perturbed parameter point, run CLASS through CLASSProblem with matching settings (T_cmb, Omega_b, Omega_cdm, l_max_g/l_max_pol_g = lmax = 6, newtonian gauge, ad initial conditions, w0_fld = -0.9, wa_fld = 0.1, cs2_fld = 0.9, use_ppf = no) and compute central (not one-sided) finite-difference Jacobians with FiniteDiff.jl. Evaluate the CMB D_l over ls = 25:3000 and matter power spectrum over ks = 10^0.5–3.5 (100 log-spaced points), then interpolate CLASS derivative values onto the common k/ℓ grid with DataInterpolations LinearInterpolation before differencing against the ForwardDiff Jacobians. Compare with plot_compare at tol = 7e-11 and judge success against the paper's reported agreement level (spectra agree with established codes up to 0.1% with standard precision levels), asserting absolute tolerances for the ∂D_l/∂p comparisons.

## Related tools

- **FiniteDiff.jl** (computes the central finite-difference Jacobians from CLASS outputs at perturbed parameter values) — github.com/hersle/SymBoltz.jl
- **CLASS** (reference Einstein-Boltzmann solver evaluated at perturbed parameter values via CLASSProblem to produce finite-difference derivative targets) — github.com/lesgourg/class_public
- **ForwardDiff.jl** (provides the automatic-differentiation Jacobians being validated against the finite-difference reference) — github.com/hersle/SymBoltz.jl
- **DiffResults.jl** (evaluates values and derivatives together via DiffResults.jacobian!)
- **DataInterpolations** (LinearInterpolation of CLASS derivative values onto the common k/ℓ grid before comparison)
- **Plots** (plot_compare visualization of the derivative-agreement comparison)

## Examples

```
using ForwardDiff, FiniteDiff; dlgP_dlgθs = ForwardDiff.jacobian(lgP, log10.(θ))
```

## Evaluation signals

- AD and finite-difference Jacobians of D_l(TT) agree within tol = 7e-11 in the plot_compare comparison
- No asserted absolute tolerance for the ∂D_l/∂p comparisons is violated
- Relative/absolute differences are computed on a common grid after LinearInterpolation, not on mismatched native grids
- CLASS runs use matching settings to SymBoltz: l_max_g/l_max_pol_g = lmax = 6, newtonian gauge, ad initial conditions, w0_fld = -0.9, wa_fld = 0.1, cs2_fld = 0.9, use_ppf = no
- Overall agreement is consistent with the paper's reported level: spectra agree with established codes up to 0.1% with standard precision levels

## Limitations

- Central finite differences only approximate derivatives to finite step-size accuracy, whereas AD gives exact derivatives of any output with respect to any input
- The comparison requires interpolating CLASS derivatives onto the SymBoltz k/ℓ grid, introducing interpolation error
- Fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented in SymBoltz, so this workflow is limited to forward-mode Jacobians

## Evidence

- [methods] SymBoltz's ForwardDiff.jacobian derivatives of the CMB D_l(TT) with respect to the varied parameters are directly compared against the CLASS finite-difference Jacobians using plot_compare at tolerance tol = 7e-11.: [evidence span withheld — anchor status: not_found]
- [abstract] Output spectra agree with established codes up to 0.1% with standard precision levels.: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [methods] Run CLASS via CLASSProblem for w0waCDM comparison: [evidence span withheld — anchor status: not_found]
- [intro] using ForwardDiff, FiniteDiff: "using ForwardDiff, FiniteDiff"
