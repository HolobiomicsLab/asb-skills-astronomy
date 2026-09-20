---
name: einstein-boltzmann-solver-cross-validation
description: 'Use when a new or modified cosmological solver must be validated against
  CLASS: compare background quantities, thermodynamics, spectra, or parameter derivatives
  by quantifying maximum differences against stated tolerances.'
when_to_use_negative:
- When validating against a code other than CLASS for which matched settings (gauge,
  recombination, reionization, initial conditions) cannot be reproduced — alignment
  mismatch would dominate the error
- When the goal is reverse-mode AD of scalar loss functions, which SymBoltz does not
  yet implement (the article lists it as missing)
- When comparing quantities that require approximation switching schemes (TCA, UFA,
  RSA), which SymBoltz deliberately omits and which would bias the cross-comparison
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SymBoltz
  role: Symbolic-numeric solver under validation; models built with ΛCDM()/w0waCDM
    and solved via CosmologyProblem with jac = true, sparse = true
  repo: https://github.com/hersle/SymBoltz.jl
- name: CLASS
  role: Reference Einstein–Boltzmann solver run through CLASSProblem with matched
    precision settings to produce ground-truth outputs and finite-difference Jacobians
  repo: https://github.com/lesgourg/class_public
- name: ForwardDiff.jl
  role: Computes automatic-differentiation Jacobians of CMB spectra D_l with respect
    to (h, Ω_c, Ω_b) for derivative cross-validation
- name: FiniteDiff.jl
  role: Builds central finite-difference Jacobians from CLASS runs at perturbed parameter
    values
- name: CairoMakie
  role: Renders plot_compare layouts (main panel + error panel) saved as class_comparison.png
- name: Plots
  role: Alternative plotting backend for derivative-agreement comparisons
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:b08dab6ddf9cf4bec3607ff92f12464abe7fd99a7d190c3bb6ca747fa414780a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - CLASS
  - CairoMakie
  - DiffResults.jl
  - FiniteDiff.jl
  - ForwardDiff.jl
  - Julia
  - Plots
  - SymBoltz
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# einstein-boltzmann-solver-cross-validation

## Summary

Cross-validate a linear Einstein–Boltzmann solver (SymBoltz) against an established reference code (CLASS) by solving an identical cosmology in both, aligning outputs on a common grid, and computing per-quantity relative/absolute errors against documented tolerances.

## When to use

Use when a new or modified cosmological solver must be validated against CLASS: compare background quantities, thermodynamics, spectra, or parameter derivatives by quantifying maximum differences against stated tolerances.

## When NOT to use

- When validating against a code other than CLASS for which matched settings (gauge, recombination, reionization, initial conditions) cannot be reproduced — alignment mismatch would dominate the error
- When the goal is reverse-mode AD of scalar loss functions, which SymBoltz does not yet implement (the article lists it as missing)
- When comparing quantities that require approximation switching schemes (TCA, UFA, RSA), which SymBoltz deliberately omits and which would bias the cross-comparison

## Inputs

- Cosmological model definition in SymBoltz symbolic-numeric format (ΛCDM() or w0waCDM components)
- Parameter set (e.g. parameters_Planck18; h, Ω_c0, Ω_b0, T_cmb, w0_fld/wa_fld/cs2_fld)
- Wavenumber grid ks (e.g. 10^0.5–3.5, 100 log-spaced points) and multipole range ls (e.g. 25:3000)
- CLASS reference outputs from CLASSProblem (background, thermodynamics, spectra, finite-difference Jacobians)

## Outputs

- Per-quantity maximum relative or absolute error values (maxerr) versus documented tolerances
- Comparison plots (class_comparison.png) with CLASS solid vs SymBoltz dashed curves and an error panel
- Numeric derivative-agreement tables comparing ForwardDiff Jacobians to CLASS finite-difference Jacobians

## How to apply

Build the model in SymBoltz (e.g. M = ΛCDM() or w0waCDM with w0 = -0.9, wa = 0.1, c_s^2 = 0.9), instantiate Planck18 parameters, and compile with CosmologyProblem(M, p; jac = true, sparse = true) to exploit the analytical sparse Jacobian. Solve the identical cosmology in CLASS via CLASSProblem with matched settings (recfast_Hswitch = 1, recfast_Heswitch = 6, reio_camb reionization, Newtonian gauge, adiabatic initial conditions, l_max_g = l_max_pol_g = lmax, use_ppf = no for PPF-off w0waCDM). Extract matched quantities (τ, H, density components, sound horizon, Xe, Tb, D_l(TT/EE/TE), P(k)) from both solvers and align them on a common grid with DataInterpolations LinearInterpolation using linear extrapolation. Compute differences as relative error (SymBoltz/CLASS − 1) or absolute error where a quantity crosses zero, and record the maximum error per quantity. Validate each quantity's maxerr against the paper's tolerances: tol ≈ 6e-3 for background quantities, tol = 8e-2 for the photon-baryon sound horizon, tol = 7e-11 for ∂D_l/∂p derivatives, and ~0.1% overall spectra agreement. Render per-quantity comparisons with plot_compare (CLASS solid vs SymBoltz dashed, lower error panel with symmetric limits from maxerr) using CairoMakie or Plots.

## Related tools

- **SymBoltz** (Symbolic-numeric solver under validation; models built with ΛCDM()/w0waCDM and solved via CosmologyProblem with jac = true, sparse = true) — https://github.com/hersle/SymBoltz.jl
- **CLASS** (Reference Einstein–Boltzmann solver run through CLASSProblem with matched precision settings to produce ground-truth outputs and finite-difference Jacobians) — https://github.com/lesgourg/class_public
- **ForwardDiff.jl** (Computes automatic-differentiation Jacobians of CMB spectra D_l with respect to (h, Ω_c, Ω_b) for derivative cross-validation)
- **FiniteDiff.jl** (Builds central finite-difference Jacobians from CLASS runs at perturbed parameter values)
- **CairoMakie** (Renders plot_compare layouts (main panel + error panel) saved as class_comparison.png)
- **Plots** (Alternative plotting backend for derivative-agreement comparisons)

## Examples

```
prob = CosmologyProblem(M, p; jac = true, sparse = true); sol = solve(prob, ks); plot_compare(a1, a2, τ1, τ2, "a", "τ"; tol = 6e-3)
```

## Evaluation signals

- Background-quantity maxerr stays within tol ≈ 6e-3 as used in the paper's plot_compare calls
- Photon-baryon sound horizon maxerr stays within tol = 8e-2
- AD vs finite-difference ∂D_l/∂p comparisons pass the asserted absolute tolerance tol = 7e-11
- Overall output spectra agree with CLASS up to ~0.1% at standard precision levels
- Error panels in plot_compare show symmetric axis limits set from the recorded maximum error, with no quantity exceeding its threshold

## Limitations

- Agreement tolerances are quantity-specific (6e-3 background, 8e-2 sound horizon, 7e-11 derivatives); a single global threshold is inappropriate
- Fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented in SymBoltz
- The CLASS run must reproduce matching settings exactly (recfast_Hswitch = 1, recfast_Heswitch = 6, reio_camb, Newtonian gauge, adiabatic initial conditions, l_max_g = l_max_pol_g = lmax) or differences reflect configuration, not solver accuracy
- Quantities crossing zero require absolute rather than relative error to avoid division blow-up

## Evidence

- [abstract] Output spectra agree with established codes up to 0.1% with standard precision levels.: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [intro] plot_compare tolerance for background quantities: "plot_compare(a1, a2, τ1, τ2, "a", "τ"; tol = 6e-3)"
- [methods] Problem compilation with analytical sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [methods] CLASSProblem configuration for w0waCDM comparison: "prob = CLASSProblem(
        "write_background" => "yes","
- [methods] ForwardDiff Jacobian of log power spectrum: "dlgP_dlgθs = ForwardDiff.jacobian(lgP, log10.(θ))"
