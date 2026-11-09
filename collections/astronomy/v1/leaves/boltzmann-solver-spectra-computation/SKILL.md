---
name: boltzmann-solver-spectra-computation
description: Use when a w0waCDM/ΛCDM model with given cosmological parameters must
  be solved for the linear Einstein–Boltzmann equations to produce P(k) and CMB D_l
  spectra, especially when a CLASS cross-check or differentiable outputs are needed.
when_to_use_negative:
- Nonlinear or non-linear-regime power spectra are required — SymBoltz solves only
  the linear Einstein–Boltzmann equations
- The target analysis requires fast reverse-mode automatic differentiation of scalar
  loss functions, which is not yet implemented in SymBoltz
- The model relies on approximation-switching schemes (TCA, UFA, RSA) semantics rather
  than approximation-free full-equation solutions
tools:
- name: SymBoltz
  role: 'Primary solver: symbolic-numeric specification and integration of the linear
    Einstein–Boltzmann equations; provides CosmologyProblem, spectrum_matter and spectrum_cmb'
  repo: https://github.com/hersle/SymBoltz.jl
- name: Julia
  role: Implementation language; run with multiple threads since SymBoltz parallelizes
    integration of different perturbation k-modes with multithreading
- name: CLASS
  role: Reference Einstein–Boltzmann solver used for the cross-validation comparison
    via CLASSProblem
  repo: https://github.com/lesgourg/class_public/
- name: Plots
  role: Plotting overlay and residual panels for the spectra-agreement figures
- name: OrdinaryDiffEq.jl
  role: Efficient high-order implicit ODE solvers used to integrate the stiff full
    equations without approximation switching
  repo: https://github.com/SciML/OrdinaryDiffEq.jl
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:86d736fcaa371fcb4f0812beaa478f693d07cf60f955176f4c23d4c8e95e165c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - CLASS
  - CairoMakie
  - DataInterpolations
  - Julia
  - OrdinaryDiffEq.jl
  - Plots
  - SymBoltz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# boltzmann-solver-spectra-computation

## Summary

Compute cosmological power spectra (matter P(k) and CMB TT/TE/EE D_l spectra) by solving the linear Einstein–Boltzmann equations with the symbolic-numeric Julia package SymBoltz, and validate results against the established code CLASS at ~0.1% relative agreement.

## When to use

Use when a w0waCDM/ΛCDM model with given cosmological parameters must be solved for the linear Einstein–Boltzmann equations to produce P(k) and CMB D_l spectra, especially when a CLASS cross-check or differentiable outputs are needed.

## When NOT to use

- Nonlinear or non-linear-regime power spectra are required — SymBoltz solves only the linear Einstein–Boltzmann equations
- The target analysis requires fast reverse-mode automatic differentiation of scalar loss functions, which is not yet implemented in SymBoltz
- The model relies on approximation-switching schemes (TCA, UFA, RSA) semantics rather than approximation-free full-equation solutions

## Inputs

- Cosmological model definition (symbolic SymBoltz model, e.g. w0waCDM) with parameter values (Planck18 base plus w0 = -0.9, wa = 0.1, c_s^2 = 0.9)
- Log-spaced grid of perturbation wavenumbers ks (e.g. [4e0, 4e1, 4e2, 4e3])
- Multipole range l = 25:3000 with SphericalBesselCache(25:25:3000)
- CLASS configuration (output 'mPk, tCl, pCl, lCl', Newtonian gauge, adiabatic scalar modes, recfast) for the comparison run

## Outputs

- Matter power spectrum P(k) array from spectrum_matter
- CMB D_l spectra (TT, TE, EE, and optionally ψψ, ψT, ψE) from spectrum_cmb with normalization = :Dl
- Overlay plots with residual panels (relative ratio SymBoltz/CLASS - 1 or absolute differences) for the spectra-agreement figure/table
- Compiled CosmologyProblem solution object supporting differentiation of outputs with respect to inputs

## How to apply

Define the cosmological model symbolically in SymBoltz (e.g. w0waCDM with lmax = 6, reionization enabled, Hswitch = 1, Heswitch = 6) with Planck18 parameters plus w0 = -0.9, wa = 0.1 and c_s^2 = 0.9, then compile it into a numerical problem via CosmologyProblem (with analytical sparse Jacobian, jac = true, sparse = true). Solve the model on a log-spaced grid of wavenumbers ks (e.g. ks = [4e0, 4e1, 4e2, 4e3] in the paper, extended to log-spaced grids for the comparison) using the implicit stiff ODE solvers from OrdinaryDiffEq.jl, and compute the matter power spectrum with spectrum_matter and the CMB D_l spectra with spectrum_cmb using SphericalBesselCache(25:25:3000), l = 25:3000 and normalization = :Dl. For validation, run CLASS via CLASSProblem with output 'mPk, tCl, pCl, lCl' in the Newtonian gauge, adiabatic scalar modes and recfast recombination, using identical parameter values, then interpolate CLASS results onto the SymBoltz grid with LinearInterpolation and compute residuals (relative ratio SymBoltz/CLASS - 1, or absolute difference where spectra cross zero) with plot_compare. Rationale: SymBoltz solves the full stiff equations at all times without TCA/UFA/RSA approximation switching, so agreement to within ~0.1% against CLASS at standard precision is the expected outcome and serves as the acceptance criterion.

## Related tools

- **SymBoltz** (Primary solver: symbolic-numeric specification and integration of the linear Einstein–Boltzmann equations; provides CosmologyProblem, spectrum_matter and spectrum_cmb) — https://github.com/hersle/SymBoltz.jl
- **Julia** (Implementation language; run with multiple threads since SymBoltz parallelizes integration of different perturbation k-modes with multithreading)
- **CLASS** (Reference Einstein–Boltzmann solver used for the cross-validation comparison via CLASSProblem) — https://github.com/lesgourg/class_public/
- **Plots** (Plotting overlay and residual panels for the spectra-agreement figures)
- **OrdinaryDiffEq.jl** (Efficient high-order implicit ODE solvers used to integrate the stiff full equations without approximation switching) — https://github.com/SciML/OrdinaryDiffEq.jl

## Examples

```
prob = CosmologyProblem(M, p; jac = true, sparse = true); sol = solve(prob, ks); Ps = spectrum_matter(modes, prob, ks)
```

## Evaluation signals

- SymBoltz spectra agree with CLASS to within ~0.1% relative difference (relative ratio SymBoltz/CLASS - 1) at standard precision levels
- Where CMB spectra cross zero, use absolute differences instead of relative ratios in the residual panel
- plot_compare tolerance assertion on maximum absolute difference per spectrum passes
- Compiled problem uses analytical sparse Jacobian (jac = true, sparse = true) so the background and perturbation stages are separated and interpolation of arbitrary variables from the solution works
- Solver accuracy can be sanity-checked via a work-precision diagram against a low-tolerance reference solution

## Limitations

- More work is needed to implement additional features and fast reverse-mode automatic differentiation of scalar loss functions
- Only linear perturbation theory is covered (no nonlinear evolution)
- CLASS compilation may need Makefile adaptation (compiler name, optimization flags such as -O4 -ffast-math, optional -fopenmp requiring gcc >= 4.2)
- When comparing spectra, relative residuals are ill-defined where spectra cross zero — absolute differences must be substituted

## Evidence

- [abstract] Output spectra agree with established codes up to 0.1% with standard precision levels.: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [methods] prob = CosmologyProblem(M, p; jac = true, sparse = true): "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [methods] Ps = spectrum_matter(modes, prob, ks): [evidence span withheld — anchor status: not_found]
- [methods] Dls = spectrum_cmb(modes, prob, jl, ls; normalization = :Dl): [evidence span withheld — anchor status: not_found]
- [abstract] Implicit solvers integrate the full stiff equations at all times without approximations.: "Implicit solvers integrate the full stiff equations at all times without approximations, which greatly simplifies the code."
- [readme] SymBoltz.jl is a Julia package for solving the linear Einstein-Boltzmann equations in cosmology.: "SymBoltz.jl is a Julia package for solving the linear Einstein-Boltzmann equations in cosmology. It is:"
