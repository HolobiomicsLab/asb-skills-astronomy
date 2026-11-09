---
name: relative-error-quantification-between-codes
description: Use when validating that a new solver reproduces a reference code's outputs
  for matched physical quantities on a common grid, and you need per-quantity maximum-error
  metrics against documented tolerances.
when_to_use_negative:
- When the two codes cannot be run on an identical parameter set and matched output
  grid — interpolated misalignment would dominate the error metric
- When quantities cross zero, where relative error is ill-defined; switch to absolute
  error rather than reporting a meaningless ratio
- When comparing approximation-switching codes against each other, where tolerance
  differences may reflect approximation schemes (TCA/UFA/RSA) rather than solver accuracy
tools:
- name: SymBoltz
  role: Solver under test; produces background and thermodynamics quantities (τ, H,
    densities, sound horizon, Xe, Tb, sound speed) to compare against CLASS
  repo: https://github.com/hersle/SymBoltz.jl
- name: CLASS
  role: Reference Einstein-Boltzmann solver invoked via CLASSProblem to generate baseline
    quantities for the error computation
  repo: https://github.com/lesgourg/class_public/
- name: CairoMakie
  role: Renders the plot_compare layout (main comparison panel + lower error panel
    with symmetric limits from maxerr) saved as class_comparison.png
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:40faf7af16501e0f620058bacdb77850336f9d3a865f977aaa11b2eb49e3155a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - CLASS
  - CairoMakie
  - Julia
  - Plots
  - SymBoltz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# relative-error-quantification-between-codes

## Summary

A per-quantity validation technique that quantifies agreement between two numerical solvers (here SymBoltz vs CLASS) by computing relative errors (SymBoltz/CLASS − 1) or absolute errors for zero-crossing quantities, recording the maximum error per quantity, and checking it against an explicitly documented tolerance (e.g. tol = 6e-3 for background, tol = 8e-2 for the sound horizon, 0.1% for spectra).

## When to use

Use when validating that a new solver reproduces a reference code's outputs for matched physical quantities on a common grid, and you need per-quantity maximum-error metrics against documented tolerances.

## When NOT to use

- When the two codes cannot be run on an identical parameter set and matched output grid — interpolated misalignment would dominate the error metric
- When quantities cross zero, where relative error is ill-defined; switch to absolute error rather than reporting a meaningless ratio
- When comparing approximation-switching codes against each other, where tolerance differences may reflect approximation schemes (TCA/UFA/RSA) rather than solver accuracy

## Inputs

- Solution objects from the two solvers (e.g. SymBoltz sol2 = solve(prob, k) and a CLASSProblem solution for the same cosmology)
- Matched physical quantity time series: conformal time τ, Hubble function H, density components, equations of state, sound horizon, luminosity distance, optical-depth derivatives, free electron fraction Xe, baryon temperature Tb, baryon sound speed
- Per-quantity agreement tolerances (e.g. tol = 6e-3 for background quantities, tol = 8e-2 for the sound horizon)

## Outputs

- Per-quantity maximum-error values (maxerr) as relative error (SymBoltz/CLASS − 1) or absolute differences
- Pass/fail validation against the documented per-quantity tolerance
- A two-panel comparison plot (class_comparison.png via CairoMakie): main panel CLASS solid vs SymBoltz dashed, lower panel the error with symmetric limits from maxerr

## How to apply

Run both codes on the identical cosmology (e.g. build the ΛCDM model with parameters_Planck18, compile with CosmologyProblem(jac = true, sparse = true), and configure CLASS via CLASSProblem with recfast_Hswitch = 1, recfast_Heswitch = 6, reio_camb reionization, Newtonian gauge and adiabatic initial conditions). Extract matched quantities (τ, H, density components, equations of state, sound horizon, luminosity distance, optical-depth derivatives, Xe, Tb, baryon sound speed) from both solvers and align them on a common grid using LinearInterpolation with linear extrapolation. Compute per-quantity differences as relative error (SymBoltz/CLASS − 1), or absolute error (SymBoltz − CLASS) for quantities crossing zero, and record the maximum error per quantity. Validate by checking each maxerr against the documented threshold — tol ≈ 6e-3 for background quantities in plot_compare calls, tol = 8e-2 for the photon-baryon sound horizon, and 0.1% for output spectra. Render the comparison with plot_compare (CLASS solid vs SymBoltz dashed in the main panel, error in the lower panel with symmetric error-axis limits set from the maximum error) using CairoMakie.

## Related tools

- **SymBoltz** (Solver under test; produces background and thermodynamics quantities (τ, H, densities, sound horizon, Xe, Tb, sound speed) to compare against CLASS) — https://github.com/hersle/SymBoltz.jl
- **CLASS** (Reference Einstein-Boltzmann solver invoked via CLASSProblem to generate baseline quantities for the error computation) — https://github.com/lesgourg/class_public/
- **CairoMakie** (Renders the plot_compare layout (main comparison panel + lower error panel with symmetric limits from maxerr) saved as class_comparison.png)

## Examples

```
plot_compare(a1, a2, τ1, τ2, "a", "τ"; tol = 6e-3)
```

## Evaluation signals

- Each quantity's maximum difference stays within its documented tolerance: maxerr ≤ tol ≈ 6e-3 for background quantities, ≤ 8e-2 for the photon-baryon sound horizon
- Output spectra agreement at the 0.1% level with standard precision levels
- Error-axis limits in the plot are symmetric and set from the recorded maximum error per quantity
- Quantities crossing zero are evaluated with absolute error (SymBoltz − CLASS) rather than relative error
- Both solvers were configured for the identical cosmology (Planck18 parameters, recfast recombination with recfast_Hswitch = 1 and recfast_Heswitch = 6, reio_camb, Newtonian gauge, adiabatic initial conditions)

## Limitations

- The tolerances are per-quantity and not uniform: background quantities use tol ≈ 6e-3 while the sound horizon uses tol = 8e-2, so a single global threshold is inappropriate
- The 0.1% spectra agreement holds only 'with standard precision levels'; tighter tolerances in either solver would change the maxerr
- Relative error is undefined or unstable for quantities crossing zero, requiring fallback to absolute error

## Evidence

- [abstract] Spectra agreement tolerance: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [intro] plot_compare tolerance example: "plot_compare(a1, a2, τ1, τ2, "a", "τ"; tol = 6e-3)"
- [methods] Comparison workflow: [evidence span withheld — anchor status: not_found]
- [readme] Package identity: "SymBoltz.jl is a Julia package for solving the linear Einstein-Boltzmann equations in cosmology."
