---
name: einstein-boltzmann-perturbation-evolution
description: Use when you need perturbation-level cosmological observables — metric
  potentials Φ/Ψ, density overdensities δ, velocity divergences θ, shear and polarization
  moments — as functions of conformal time for fixed wavenumbers, or when validating
  a new EB solver against CLASS.
when_to_use_negative:
- You only need background quantities or power spectra (P(k), C_l) without perturbation-level
  cross-validation — use the spectrum_matter/spectrum_cmb workflows instead
- You require fast reverse-mode automatic differentiation of scalar loss functions
  — this is not yet implemented in SymBoltz
- Your model requires approximation-switching schemes or features SymBoltz does not
  yet implement (the authors note more work is needed for additional features)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3298
tools:
- name: SymBoltz
  role: Symbolic-numeric Julia package that specifies the w0waCDM model, compiles
    it via CosmologyProblem with analytical sparse Jacobians, and integrates the full
    Einstein–Boltzmann equations for each k-mode without approximations
  repo: github.com/hersle/SymBoltz.jl
- name: CLASS
  role: Reference Einstein–Boltzmann solver run via CLASSProblem with matched gauge,
    primordial mode, multilevel, recombination (Hswitch = 1, Heswitch = 6) and fluid
    (w0_fld, wa_fld, cs2_fld, use_ppf = 'no') settings for cross-validation
  repo: github.com/lesgourg/class_public
- name: DataInterpolations
  role: LinearInterpolation with ExtrapolationType.Linear to align CLASS perturbation
    series onto SymBoltz conformal-time gridpoints over the common time range
- name: CairoMakie
  role: 'Renders the comparison layout: value plots plus error panels showing relative
    difference SymBoltz/CLASS − 1 (or absolute difference) with symmetric y-range
    from the maximum error'
- name: OrdinaryDiffEq.jl
  role: Provides the efficient high-order implicit ODE solvers that integrate the
    full stiff equations at all times without approximation switches
  repo: github.com/SciML/OrdinaryDiffEq.jl
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:02a2b60351bdd7f0f1282826ce27cadb25e281d9b2502df8b3648c48f14b93f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CLASS
  - CairoMakie
  - DataInterpolations
  - Julia
  - OrdinaryDiffEq.jl
  - SymBoltz
  - Unitful
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3298
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# einstein-boltzmann-perturbation-evolution

## Summary

Integrating the linear Einstein–Boltzmann equations for individual perturbation k-modes and validating the solution against an established Boltzmann solver (CLASS). SymBoltz solves the full stiff equations without approximation switches (TCA, UFA, RSA), so cross-code agreement of gauge-invariant perturbation variables is the key accuracy check.

## When to use

Use when you need perturbation-level cosmological observables — metric potentials Φ/Ψ, density overdensities δ, velocity divergences θ, shear and polarization moments — as functions of conformal time for fixed wavenumbers, or when validating a new EB solver against CLASS.

## When NOT to use

- You only need background quantities or power spectra (P(k), C_l) without perturbation-level cross-validation — use the spectrum_matter/spectrum_cmb workflows instead
- You require fast reverse-mode automatic differentiation of scalar loss functions — this is not yet implemented in SymBoltz
- Your model requires approximation-switching schemes or features SymBoltz does not yet implement (the authors note more work is needed for additional features)

## Inputs

- Cosmological parameter set (e.g. Planck18 with w0 = -0.9, wa = 0.1, cₛ² = 0.9 for w0waCDM)
- Perturbation wavenumber(s) k (e.g. k = 1e1/Mpc; SymBoltz examples use ks = [4e0, 4e1, 4e2, 4e3])
- Multilevel truncation settings (lmax = 6 on the SymBoltz side; l_max_g/l_max_pol_g/l_max_ur/l_max_ncdm = 6 in CLASS)

## Outputs

- Conformal-time series of perturbation variables (Φ, Ψ, δ, θ, shear, polarization moments) from SymBoltz and CLASS
- Per-variable maximum relative-difference (or absolute-difference for zero-crossing variables) values with pass/fail against a tolerance such as 6e-3
- Comparison plots (values and error panels with symmetric y-range from the maximum error)

## How to apply

Build the SymBoltz w0waCDM model with lmax = 6 and compile it with CosmologyProblem (analytical + sparse Jacobian, e.g. jac = true, sparse = true) using Planck18 parameters overridden with w0 = -0.9, wa = 0.1 and cₛ² = 0.9. Configure the CLASS reference through CLASSProblem with matched settings: Newtonian gauge, adiabatic 'ad' scalar modes, l_max_g/l_max_pol_g/l_max_ur/l_max_ncdm = 6, recfast recombination with Hswitch = 1 and Heswitch = 6, reio_camb reionization, w0_fld = -0.9, wa_fld = 0.1, cs2_fld = 0.9, use_ppf = 'no', and k_output_values set to the comparison wavenumber (k = 1e1/Mpc). Solve both problems for the same k-mode, then extract Φ, Ψ, δ and θ for photons, baryons, CDM, massive neutrinos and dark energy, plus shear and polarization moments, as functions of conformal time. Restrict to the common time range and align CLASS series onto SymBoltz' times with DataInterpolations.LinearInterpolation (extrapolation = ExtrapolationType.Linear). Validate with plot_compare by printing and asserting the maximum relative difference per variable (SymBoltz/CLASS − 1, switching to absolute difference SymBoltz − CLASS for zero-crossing variables) within a tolerance such as tol = 6e-3, consistent with the paper's stated 0.1% spectral agreement.

## Related tools

- **SymBoltz** (Symbolic-numeric Julia package that specifies the w0waCDM model, compiles it via CosmologyProblem with analytical sparse Jacobians, and integrates the full Einstein–Boltzmann equations for each k-mode without approximations) — github.com/hersle/SymBoltz.jl
- **CLASS** (Reference Einstein–Boltzmann solver run via CLASSProblem with matched gauge, primordial mode, multilevel, recombination (Hswitch = 1, Heswitch = 6) and fluid (w0_fld, wa_fld, cs2_fld, use_ppf = 'no') settings for cross-validation) — github.com/lesgourg/class_public
- **DataInterpolations** (LinearInterpolation with ExtrapolationType.Linear to align CLASS perturbation series onto SymBoltz conformal-time gridpoints over the common time range)
- **CairoMakie** (Renders the comparison layout: value plots plus error panels showing relative difference SymBoltz/CLASS − 1 (or absolute difference) with symmetric y-range from the maximum error)
- **OrdinaryDiffEq.jl** (Provides the efficient high-order implicit ODE solvers that integrate the full stiff equations at all times without approximation switches) — github.com/SciML/OrdinaryDiffEq.jl

## Examples

```
prob = CosmologyProblem(M, p; jac = true, sparse = true); ks = [4e0, 4e1, 4e2, 4e3]; sol = solve(prob, ks)
```

## Evaluation signals

- Maximum relative difference of k=0 metric potentials Φ and Ψ against CLASS stays within 3e-4 over scale factor a
- Per-variable maximum relative difference printed and asserted inside plot_compare passes the tolerance (e.g. tol = 6e-3)
- Overall output spectra agree with established codes up to 0.1% with standard precision levels
- CLASS settings are matched one-to-one: Newtonian gauge, 'ad' scalar modes, l_max = 6 everywhere, Hswitch = 1, Heswitch = 6, reio_camb, use_ppf = 'no', w0_fld/wa_fld/cs2_fld equal the SymBoltz overrides
- Both solutions are compared only on the common conformal-time range after linear interpolation alignment, and zero-crossing variables use absolute rather than relative difference

## Limitations

- SymBoltz does not yet implement fast reverse-mode automatic differentiation of scalar loss functions
- More work is needed to implement additional features beyond the current component/model library
- Multilevel truncation at lmax = 6 and the chosen k-mode limit the scope of the perturbation comparison; agreement is demonstrated per fixed k (e.g. k = 1e1/Mpc) rather than across the full spectrum at once
- Solver performance, while comparable to approximation-based codes, depends on running Julia with multiple threads and optionally swapping BLAS backends via MKL.jl (Intel) or AppleAccelerate.jl (Macs)

## Evidence

- [abstract] Approximation-free EB solving without switches: "Implicit solvers integrate the full stiff equations at all times without approximations, which greatly simplifies the code."
- [abstract] Agreement level with established codes: "Output spectra agree with established codes up to 0.1% with standard precision levels."
- [intro] Comparison tolerance in plots: "plot_compare(a1, a2, τ1, τ2, "a", "τ"; tol = 6e-3)"
- [methods] Compiling the problem with sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [methods] Running CLASS via CLASSProblem: "prob = CLASSProblem(
        "write_background" => "yes","
- [methods] Solving a set of k-modes: "ks = [4e0, 4e1, 4e2, 4e3]
sol = solve(prob, ks)"
- [methods] CLASS repository as reference solver: "the fantastic Einstein-Boltzmann solver [CLASS](https://github.com/lesgourg/class_public/)"
- [methods] Multithreaded k-mode integration: "SymBoltz parallelizes integration of different perturbation $k$-modes with multithreading by default."
- [intro] Approximation-free objective listing TCA/UFA/RSA: "**Approximation-free:** Full equations are solved at all times without tight-coupling, ultrarelativistic fluid and radiation-streaming approximations (TCA, UFA and RSA) using efficient implicit ODE"
