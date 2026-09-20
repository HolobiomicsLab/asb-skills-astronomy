---
name: cmb-spectrum-computation
description: Use when you need linear CMB angular power spectra (TT/EE/TE D_l) from
  a cosmological model, especially when the spectra must be differentiable (ForwardDiff)
  or compared against CLASS/established Boltzmann codes.
when_to_use_negative:
- You only need the matter power spectrum P(k) over ks = 10^0.5–3.5 — use spectrum_matter
  instead of spectrum_cmb
- You need non-linear or lensed spectra, or fast reverse-mode AD of a scalar loss
  function — the paper states more work is needed for fast reverse-mode automatic
  differentiation and additional features
- A model component is not implemented in SymBoltz's src/models/ library and cannot
  be adapted from existing components
edam_operation: http://edamontology.org/operation_2426
tools:
- name: SymBoltz
  role: Symbolic-numeric Julia package that solves the linear Einstein–Boltzmann equations
    and computes CMB spectra via spectrum_cmb
  repo: https://github.com/hersle/SymBoltz.jl
- name: CLASS
  role: Reference Einstein–Boltzmann solver run through CLASSProblem to build finite-difference
    Jacobians for cross-validation
  repo: https://github.com/lesgourg/class_public
- name: ForwardDiff.jl
  role: Forward-mode automatic differentiation used to obtain exact derivatives of
    the CMB spectra with respect to (h, Ω_c0, Ω_b0)
- name: FiniteDiff.jl
  role: Central finite-difference Jacobians from CLASS spectra at perturbed parameter
    values
- name: Plots
  role: Plotting the derivative-agreement comparison of SymBoltz and CLASS spectra/derivatives
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:1bc22b6c4e0c8913925c5abf7bb18bb5e1c4b34d4c69dda1ed473a2dcf27731e
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cmb-spectrum-computation

## Summary

Compute CMB temperature and polarization power spectra (D_l for TT, EE, TE) with SymBoltz by solving the approximation-free linear Einstein–Boltzmann equations and projecting with a cached spherical Bessel transform, enabling differentiable spectra validated against CLASS at the 0.1% level.

## When to use

Use when you need linear CMB angular power spectra (TT/EE/TE D_l) from a cosmological model, especially when the spectra must be differentiable (ForwardDiff) or compared against CLASS/established Boltzmann codes.

## When NOT to use

- You only need the matter power spectrum P(k) over ks = 10^0.5–3.5 — use spectrum_matter instead of spectrum_cmb
- You need non-linear or lensed spectra, or fast reverse-mode AD of a scalar loss function — the paper states more work is needed for fast reverse-mode automatic differentiation and additional features
- A model component is not implemented in SymBoltz's src/models/ library and cannot be adapted from existing components

## Inputs

- Symbolic SymBoltz cosmology model (e.g. w0waCDM) with Planck18-like parameters and lmax = 6
- Multipoles range ls (e.g. 25:3000) and spherical Bessel cache SphericalBesselCache(25:25:3000)
- Compiled CosmologyProblem with jac = true, sparse = true
- CLASS input settings (h, T_cmb, Omega_b, Omega_cdm, l_max_g/l_max_pol_g, w0_fld, wa_fld, cs2_fld, use_ppf = no) for cross-validation

## Outputs

- CMB angular power spectra D_l for modes TT, EE, TE (normalization = :Dl)
- ForwardDiff Jacobians ∂D_l/∂(h, Ω_c0, Ω_b0) via DiffResults.jacobian! (values and derivatives together)
- Derivative-agreement comparison plots (Plots) and numeric agreement tables vs CLASS finite differences

## How to apply

Specify a symbolic cosmology in SymBoltz (e.g. the w0waCDM model with lmax = 6, reionization on, Hswitch = 1, Heswitch = 6, Planck18-like parameters such as w0 = -0.9, wa = 0.1, c_s^2 = 0.9) and compile it with CosmologyProblem (jac = true, sparse = true for the analytical sparse Jacobian). Solve the perturbations over the requested wavenumbers, then call spectrum_cmb with modes [:TT, :EE, :TE], a spherical Bessel cache such as SphericalBesselCache(25:25:3000), the requested multipoles ls = 25:3000, and normalization = :Dl to get lensed-free linear D_l values. Because SymBoltz integrates the full stiff equations without tight-coupling, ultrarelativistic fluid or radiation-streaming approximations, the same pipeline yields ForwardDiff.jacobian derivatives of D_l with respect to parameters like (h, Ω_c0, Ω_b0) when wrapped in log-space. For validation, run CLASS via CLASSProblem with matching settings (l_max_g/l_max_pol_g = lmax, newtonian gauge, ad initial conditions, w0_fld/wa_fld/cs2_fld, use_ppf = no), interpolate onto the common ℓ grid, and compare relative/absolute differences against the paper's 0.1% agreement target using plot_compare with a tight tolerance such as tol = 7e-11 for derivative agreement.

## Related tools

- **SymBoltz** (Symbolic-numeric Julia package that solves the linear Einstein–Boltzmann equations and computes CMB spectra via spectrum_cmb) — https://github.com/hersle/SymBoltz.jl
- **CLASS** (Reference Einstein–Boltzmann solver run through CLASSProblem to build finite-difference Jacobians for cross-validation) — https://github.com/lesgourg/class_public
- **ForwardDiff.jl** (Forward-mode automatic differentiation used to obtain exact derivatives of the CMB spectra with respect to (h, Ω_c0, Ω_b0))
- **FiniteDiff.jl** (Central finite-difference Jacobians from CLASS spectra at perturbed parameter values)
- **Plots** (Plotting the derivative-agreement comparison of SymBoltz and CLASS spectra/derivatives)

## Examples

```
jl = SphericalBesselCache(25:25:3000); Dls = spectrum_cmb([:TT, :EE, :TE], prob, jl, 25:3000; normalization = :Dl)
```

## Evaluation signals

- SymBoltz CMB spectra agree with CLASS (established codes) up to 0.1% with standard precision levels
- ForwardDiff CMB D_l derivatives agree with CLASS central finite-difference Jacobians within asserted absolute tolerances (plot_compare at tol = 7e-11 for ∂D_l/∂p comparisons)
- D_l values cover the full requested multipole range (e.g. ls = 25:3000) for all three modes TT, EE, TE with normalization = :Dl
- CLASS run reproduces the SymBoltz settings: matching lmax, newtonian gauge, ad initial conditions, w0_fld/wa_fld/cs2_fld, use_ppf = no
- Interpolated CLASS derivatives onto the common ℓ grid (DataInterpolations LinearInterpolation) yield smooth relative/absolute difference curves with no check failing

## Limitations

- More work is needed to implement additional features and for fast reverse-mode automatic differentiation of scalar loss functions
- The paper reports no discussion-section content (changelog not found), so explicit limitations, missing artifacts, or reproducibility gaps beyond the abstract-level statements could not be extracted
- Agreement level (0.1%) holds only with standard precision levels; finite-difference validation requires CLASS to be run at perturbed parameter values with carefully matched settings

## Evidence

- [methods] Compute the CMB power spectrum with spectrum_cmb: "Dls = spectrum_cmb(modes, prob, jl, ls; normalization = :Dl)"
- [abstract] Approximation-free linear Boltzmann solving: "Implicit solvers integrate the full stiff equations at all times without approximations, which greatly simplifies the code."
- [abstract] 0.1% agreement target with established codes: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [abstract] Differentiable outputs via automatic differentiation: "Automatic differentiation gives exact derivatives of any output with respect to any input,"
- [readme] Symbolic compilation to fast numerical functions: "Models are specified with symbolic equations in a simple, convenient and extensible high-level format, then automatically compiled to fast numerical functions"
