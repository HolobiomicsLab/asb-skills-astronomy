---
name: cross-code-spectra-interpolation-and-residual-analysis
description: Use when a new linear perturbation solver's P(k) or CMB spectra must
  be validated against a benchmark code like CLASS, requiring grid alignment via interpolation
  and relative/absolute residual quantification.
when_to_use_negative:
- When comparing codes whose cosmological parameter sets or gauge choices differ (e.g.
  different gauge or recombination treatment) — residuals would conflate physical
  with numerical differences
- When the goal is accuracy benchmarking of a single solver's convergence (a work-precision
  diagram against a low-tolerance reference solution is the appropriate method instead)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: SymBoltz
  role: Julia package solving the linear Einstein-Boltzmann equations; provides spectrum_matter
    and spectrum_cmb to compute the spectra under test
  repo: https://github.com/hersle/SymBoltz.jl
- name: CLASS
  role: Established reference Einstein-Boltzmann solver run via CLASSProblem to produce
    benchmark P(k) and CMB spectra
  repo: https://github.com/lesgourg/class_public/
- name: DataInterpolations
  role: LinearInterpolation maps CLASS results onto the SymBoltz k-grid for like-for-like
    residuals
- name: Plots
  role: Renders overlay plots with residual panels as the spectra-agreement figure
- name: Julia
  role: Language environment in which SymBoltz, interpolation, and plotting are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:89d9cfb4cb72cd1bd564737b3889a65881a9abdcc8f596c3a2cc3094d2e2d8e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - CLASS
  - CairoMakie
  - DataInterpolations
  - Julia
  - Plots
  - SymBoltz
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cross-code-spectra-interpolation-and-residual-analysis

## Summary

Validates an Einstein-Boltzmann solver (SymBoltz) against an established reference code (CLASS) by comparing matter power spectrum P(k) and CMB D_l spectra (TT/TE/EE), interpolating one code's output onto the other's grid and computing residual statistics to quantify agreement (target ~0.1%).

## When to use

Use when a new linear perturbation solver's P(k) or CMB spectra must be validated against a benchmark code like CLASS, requiring grid alignment via interpolation and relative/absolute residual quantification.

## When NOT to use

- When comparing codes whose cosmological parameter sets or gauge choices differ (e.g. different gauge or recombination treatment) — residuals would conflate physical with numerical differences
- When the goal is accuracy benchmarking of a single solver's convergence (a work-precision diagram against a low-tolerance reference solution is the appropriate method instead)

## Inputs

- SymBoltz solved modes over a log-spaced k-grid (e.g. ks = [4e0, 4e1, 4e2, 4e3]) with identical cosmological parameters
- CLASS output for mPk, tCl, pCl, lCl computed via CLASSProblem with matched parameter values
- Multipole range specification l = 25:3000 with SphericalBesselCache(25:25:3000)

## Outputs

- Overlay plots of SymBoltz vs CLASS spectra with residual panels (spectra-agreement figure/table)
- Residual arrays: relative ratio SymBoltz/CLASS - 1, or absolute difference where spectra cross zero
- Per-spectrum maximum absolute difference statistics validated against a ~0.1% tolerance

## How to apply

Configure both codes with an identical cosmology (e.g. w0waCDM with Planck18 parameters plus w0 = -0.9, wa = 0.1, c_s^2 = 0.9; SymBoltz lmax = 6, reionization enabled, Hswitch = 1, Heswitch = 6; CLASS via CLASSProblem with output 'mPk, tCl, pCl, lCl', Newtonian gauge, adiabatic scalar modes, recfast recombination). Solve SymBoltz on a log-spaced grid of wavenumbers and compute the matter spectrum with spectrum_matter; compute CMB D_l spectra with spectrum_cmb using SphericalBesselCache(25:25:3000), l = 25:3000, normalization = :Dl. Use LinearInterpolation (DataInterpolations.jl) to map CLASS results onto the SymBoltz grid, then compute residuals as the relative ratio SymBoltz/CLASS - 1, or absolute difference where spectra cross zero, with the plot_compare routine producing overlay plots with residual panels. Enforce agreement by a tolerance assertion on the maximum absolute difference per spectrum; the article reports agreement up to 0.1% at standard precision levels.

## Related tools

- **SymBoltz** (Julia package solving the linear Einstein-Boltzmann equations; provides spectrum_matter and spectrum_cmb to compute the spectra under test) — https://github.com/hersle/SymBoltz.jl
- **CLASS** (Established reference Einstein-Boltzmann solver run via CLASSProblem to produce benchmark P(k) and CMB spectra) — https://github.com/lesgourg/class_public/
- **DataInterpolations** (LinearInterpolation maps CLASS results onto the SymBoltz k-grid for like-for-like residuals)
- **Plots** (Renders overlay plots with residual panels as the spectra-agreement figure)
- **Julia** (Language environment in which SymBoltz, interpolation, and plotting are executed)

## Examples

```
prob = CLASSProblem("write_background" => "yes", ...); Ps = spectrum_matter(modes, prob, ks); Dls = spectrum_cmb(modes, prob, jl, ls; normalization = :Dl)
```

## Evaluation signals

- Relative residual SymBoltz/CLASS - 1 stays within ~0.1% (1e-3) across all compared spectra at standard precision
- plot_compare tolerance assertion on maximum absolute difference per spectrum passes (e.g. tol = 6e-3 style check)
- Absolute-difference residuals are used (rather than ratios) in multipole/wavenumber regions where spectra cross zero
- Both codes were configured with identical cosmological parameters and matching settings (Newtonian gauge, adiabatic scalar modes, recfast recombination)

## Limitations

- Reported 0.1% agreement holds only at standard precision levels; tighter tolerances require higher precision settings in both codes
- Where CMB spectra cross zero, relative ratios are ill-defined and absolute differences must be used instead
- SymBoltz lacks fast reverse-mode automatic differentiation of scalar loss functions, and more features are still needed

## Evidence

- [abstract] agreement up to 0.1% at standard precision: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [methods] CLASS comparison via CLASSProblem: [evidence span withheld — anchor status: not_found]
- [methods] CMB spectrum computation call: "Dls = spectrum_cmb(modes, prob, jl, ls; normalization = :Dl)"
- [methods] matter spectrum computation call: "Ps = spectrum_matter(modes, prob, ks)"
- [readme] SymBoltz purpose: "SymBoltz.jl is a Julia package for solving the linear Einstein-Boltzmann equations in cosmology."
