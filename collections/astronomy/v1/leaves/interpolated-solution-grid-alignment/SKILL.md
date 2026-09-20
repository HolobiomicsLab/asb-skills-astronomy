---
name: interpolated-solution-grid-alignment
description: Use when comparing outputs of two cosmological solvers (or any two ODE
  solutions) that were sampled on different grids and you need per-quantity maximum-error
  validation against a documented tolerance.
when_to_use_negative:
- Quantities that cross zero — relative error is ill-defined there; use absolute error
  instead (as the workflow itself prescribes)
- 'Validation of nonlinear or non-linear-perturbation regimes: SymBoltz solves only
  the linear Einstein–Boltzmann equations, so comparison with nonlinear tools is out
  of scope'
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SymBoltz
  role: Source solver providing background and thermodynamics quantities to align
    and compare
  repo: https://github.com/hersle/SymBoltz.jl
- name: CLASS
  role: Reference solver (via CLASSProblem) whose quantities define the target grid
    and reference values
  repo: https://github.com/lesgourg/class_public/
- name: CairoMakie
  role: Renders the plot_compare figure with main-panel curves and lower-panel error
- name: Plots
  role: Alternative Julia plotting backend used for comparison layouts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:6bccda2b14d4e1cbbb2b80211c88d6accd62cd712440cc4a83ff182954c7118f
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Interpolated Solution Grid Alignment

## Summary

Aligns output quantities from two independent solvers (e.g. SymBoltz and CLASS) on a common grid using LinearInterpolation with linear extrapolation, then quantifies per-quantity agreement as relative error (SymBoltz/CLASS − 1) or absolute error for zero-crossing quantities.

## When to use

Use when comparing outputs of two cosmological solvers (or any two ODE solutions) that were sampled on different grids and you need per-quantity maximum-error validation against a documented tolerance.

## When NOT to use

- Quantities that cross zero — relative error is ill-defined there; use absolute error instead (as the workflow itself prescribes)
- Validation of nonlinear or non-linear-perturbation regimes: SymBoltz solves only the linear Einstein–Boltzmann equations, so comparison with nonlinear tools is out of scope

## Inputs

- SymBoltz solution object (e.g. sol2 = solve(prob, k)) for the chosen k-modes
- CLASS solution from CLASSProblem with matched cosmological parameters (Planck18)
- List of matched quantity names (τ, H, densities, sound horizon, Xe, Tb, baryon sound speed, etc.)

## Outputs

- Per-quantity maximum errors (relative or absolute) versus documented tolerances
- Comparison plot (class_comparison.png) with CLASS solid vs SymBoltz dashed curves and a lower error panel

## How to apply

Solve the same model in both solvers — e.g. SymBoltz's ΛCDM() with parameters_Planck18 compiled via CosmologyProblem(jac = true, sparse = true), and CLASS via CLASSProblem with recfast_Hswitch = 1, recfast_Heswitch = 6, reio_camb reionization, and Newtonian-gauge adiabatic initial conditions. Extract matched quantities (τ, H, density components, equations of state, sound horizon, luminosity distance, optical-depth derivatives, Xe, Tb, baryon sound speed) from each solution and map them onto a common grid using LinearInterpolation with linear extrapolation. For each quantity compute the difference as relative error (SymBoltz/CLASS − 1), falling back to absolute error (SymBoltz − CLASS) for quantities that cross zero. Record the maximum error per quantity and validate against the documented tolerances (e.g. tol ≈ 6e-3 for background/thermodynamics quantities in plot_compare calls, tol = 8e-2 for the photon-baryon sound horizon, and spectra agreement up to 0.1%). Render the comparison with plot_compare (CLASS solid vs SymBoltz dashed in the main panel, error in the lower panel with symmetric limits set from the maximum error) using CairoMakie.

## Related tools

- **SymBoltz** (Source solver providing background and thermodynamics quantities to align and compare) — https://github.com/hersle/SymBoltz.jl
- **CLASS** (Reference solver (via CLASSProblem) whose quantities define the target grid and reference values) — https://github.com/lesgourg/class_public/
- **CairoMakie** (Renders the plot_compare figure with main-panel curves and lower-panel error)
- **Plots** (Alternative Julia plotting backend used for comparison layouts)

## Examples

```
plot_compare(a1, a2, τ1, τ2, "a", "τ"; tol = 6e-3)
```

## Evaluation signals

- Each quantity's maximum error (maxerr) stays within its documented tolerance, e.g. tol ≈ 6e-3 for background quantities, tol = 8e-2 for the sound horizon
- Relative errors use the SymBoltz/CLASS − 1 convention consistently; zero-crossing quantities switch to absolute error
- Interpolation uses LinearInterpolation with linear extrapolation so both series share a common grid before differencing
- plot_compare error-axis limits are symmetric and set from the maximum error, giving a visual sanity check on maxerr
- Output spectra agreement is expected up to 0.1% at standard precision levels

## Limitations

- The sound horizon comparison tolerance is much looser (8e-2) than background-quantity tolerances (6e-3), so tolerances are quantity-specific and must not be mixed
- Linear interpolation/extrapolation on a common grid introduces interpolation error, particularly where solutions are steep (e.g. recombination-era Xe)
- SymBoltz spectra agree with established codes only up to 0.1% at standard precision — sub-permille claims require tighter settings

## Evidence

- [other] SymBoltz's photon-baryon sound horizon is compared against CLASS using plot_compare with an explicitly documented tolerance (tol = 8e-2): [evidence span withheld — anchor status: not_found]
- [methods] Workflow step on alignment via LinearInterpolation: [evidence span withheld — anchor status: not_found]
- [methods] Per-quantity error definition: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract agreement claim: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [intro] Intro research question with background tolerance: "plot_compare(a1, a2, τ1, τ2, "a", "τ"; tol = 6e-3)"
