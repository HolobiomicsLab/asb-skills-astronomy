---
name: reference-solution-error-norm-computation
description: Use when comparing implicit ODE solvers or tolerance settings on a stiff
  system (e.g. the Einstein-Boltzmann background) and you need a quantitative, solver-independent
  accuracy metric rather than trusting each solver's own tolerance claims.
when_to_use_negative:
- When the goal is agreement of physical spectra (P(k), C_l) with an established code
  like CLASS at the 0.1% level — that requires spectrum comparison, not time-grid
  L2 error
- When only a single production solve is needed and no solver/tolerance selection
  or benchmarking is intended
- When the ODE system is non-stiff and explicit methods are appropriate — implicit
  high-order solvers with sparse Jacobians would be unnecessary overhead
tools:
- name: OrdinaryDiffEq.jl
  role: Provides the implicit stiff ODE algorithms (Rodas4/5/6P, FBDF, QNDF), the
    RFLUFactorization linear-solver choice, and the retcode signaling used to flag
    failed integrations
  repo: github.com/SciML/OrdinaryDiffEq.jl
- name: SymBoltz
  role: Builds the ΛCDM model, loads Planck18 parameters, compiles the problem with
    an analytical sparse Jacobian, and exposes the background sub-problem prob.bg
    used as the benchmark system
  repo: github.com/hersle/SymBoltz.jl
- name: Plots
  role: Renders the log-log work-precision diagram (time vs L2 error) with distinct
    markers per solver
  repo: github.com/hersle/SymBoltz.jl
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:f1c08f35e34e842e79a4289c815f446b8e60a7d75dd4d3206917bcebaa1cfb9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia
  - OrdinaryDiffEq.jl
  - Plots
  - SymBoltz
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# reference-solution-error-norm-computation

## Summary

Benchmark ODE solver accuracy by solving a system once at very tight tolerances to obtain a reference solution, then quantifying each solver/tolerance configuration by the L2 norm error of its solution interpolated onto the reference time grid. This is the method behind the work-precision diagram comparing Rodas4/4P/5/5P/6P, FBDF and QNDF on SymBoltz's background equations.

## When to use

Use when comparing implicit ODE solvers or tolerance settings on a stiff system (e.g. the Einstein-Boltzmann background) and you need a quantitative, solver-independent accuracy metric rather than trusting each solver's own tolerance claims.

## When NOT to use

- When the goal is agreement of physical spectra (P(k), C_l) with an established code like CLASS at the 0.1% level — that requires spectrum comparison, not time-grid L2 error
- When only a single production solve is needed and no solver/tolerance selection or benchmarking is intended
- When the ODE system is non-stiff and explicit methods are appropriate — implicit high-order solvers with sparse Jacobians would be unnecessary overhead

## Inputs

- Compiled SymBoltz CosmologyProblem with analytical sparse Jacobian (jac=true, sparse=true) and Planck18 parameter values (parameters_Planck18)
- Background sub-problem prob.bg of the ΛCDM model
- Solver/tolerance grid: {Rodas4, Rodas5, Rodas4P, Rodas5P, Rodas6P, FBDF, QNDF} x {1e-7, 1e-8, 1e-9, 1e-10, 1e-11} abstol=reltol
- Reference configuration: Rodas5P + RFLUFactorization at abstol=reltol=1e-12

## Outputs

- Per-solver/tolerance table of (wall-time minimum over 5 runs, L2 error norm(u-ref)/length(u))
- (NaN, NaN) instability entries for solver/tolerance pairs with unsuccessful retcodes (expected for FBDF and QNDF)
- Log-log work-precision diagram (time vs error) with one curve per solver

## How to apply

First compute a trusted reference: solve the background problem (prob.bg) with Rodas5P and RFLUFactorization at abstol=reltol=1e-12. Then sweep abstol=reltol over 1/10^7 through 1/10^11 for each candidate solver (Rodas4, Rodas5, Rodas4P, Rodas5P, Rodas6P, FBDF, QNDF, all with RFLUFactorization). For each solver/tolerance pair, interpolate the solution onto the reference time grid and compute the L2 error as norm(u-ref)/length(u), so the metric is independent of solver-specific step/adaptivity. Record wall time as the minimum over N=5 repeated runs to suppress noise, and mark any failed retcode as (NaN, NaN) so instability is exposed in the diagram rather than hidden. Plot time-vs-error in log-log axes with one marked curve per solver; rationale: SymBoltz solves the full stiff equations without approximation switching, so solver choice and tolerance are the dominant accuracy controls.

## Related tools

- **OrdinaryDiffEq.jl** (Provides the implicit stiff ODE algorithms (Rodas4/5/6P, FBDF, QNDF), the RFLUFactorization linear-solver choice, and the retcode signaling used to flag failed integrations) — github.com/SciML/OrdinaryDiffEq.jl
- **SymBoltz** (Builds the ΛCDM model, loads Planck18 parameters, compiles the problem with an analytical sparse Jacobian, and exposes the background sub-problem prob.bg used as the benchmark system) — github.com/hersle/SymBoltz.jl
- **Plots** (Renders the log-log work-precision diagram (time vs L2 error) with distinct markers per solver) — github.com/hersle/SymBoltz.jl

## Examples

```
ref = solve(prob.bg, Rodas5P(), RFLUFactorization(); abstol = 1e-12, reltol = 1e-12); err = norm(u - ref)/length(u)
```

## Evaluation signals

- Reference solution retcode is success at abstol=reltol=1e-12 with Rodas5P before any comparison is made
- L2 error decreases monotonically toward the 1e-12 reference as tolerances tighten for stable solvers (Rodas4/5/4P/5P/6P)
- Failed retcodes are recorded as (NaN, NaN), and FBDF/QNDF exhibit failed integrations at several swept tolerances, matching the paper's instability record
- Each reported timing is the minimum over 5 repeated runs, not a single measurement
- Every solution is interpolated onto the reference time grid before norm(u-ref)/length(u) is computed

## Limitations

- FBDF and QNDF are unstable on this system — several swept tolerances produce failed retcodes, so their benchmark points are missing rather than merely inaccurate
- The L2 error is only as trustworthy as the 1e-12 Rodas5P reference; it is a relative, not absolute, accuracy measure
- SymBoltz still lacks fast reverse-mode automatic differentiation of scalar loss functions, so the benchmarking pipeline does not yet cover AD-through-the-solver timings
- Wall-time minima are machine-dependent (thread count, BLAS choice via MKL.jl/AppleAccelerate.jl), so absolute times are not portable across hardware

## Evidence

- [methods] This plot compares the time to solve the background vs. accuracy of the solution using different ODE solvers and tolerances.: "This plot compares the time to solve the background vs. accuracy of the solution using different ODE solvers and tolerances."
- [abstract] Implicit solvers integrate the full stiff equations at all times without approximations: "Implicit solvers integrate the full stiff equations at all times without approximations"
- [abstract] Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods.: "Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian"
- [methods] prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [readme] SymBoltz.jl is a Julia package for solving the linear Einstein-Boltzmann equations in cosmology.: "SymBoltz.jl is a Julia package for solving the linear Einstein-Boltzmann equations in cosmology."
