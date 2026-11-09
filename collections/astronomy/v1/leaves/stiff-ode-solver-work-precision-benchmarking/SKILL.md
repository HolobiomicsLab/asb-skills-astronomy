---
name: stiff-ode-solver-work-precision-benchmarking
description: Use when a stiff ODE system must be integrated with an implicit OrdinaryDiffEq.jl
  solver and you need to choose a solver and tolerance based on quantitative time-vs-accuracy
  tradeoffs rather than defaults or intuition.
when_to_use_negative:
- The problem is non-stiff and explicit solvers (e.g. Tsit5) would be appropriate;
  the swept Rodas/FBDF/QNDF implicit solvers target stiff systems like the Einstein-Boltzmann
  equations.
- The goal is absolute cross-code accuracy comparison against an external solver (e.g.
  CLASS spectra agreement within 0.1%) rather than internal solver selection.
- A single solver/tolerance has already been validated and only one run is needed;
  the 5-repeat minimum-timing protocol is only meaningful for benchmarking.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3524
tools:
- name: OrdinaryDiffEq.jl
  role: Provides the stiff implicit solvers (Rodas4/4P/5/5P/6P, FBDF, QNDF) and RFLUFactorization
    used in the tolerance sweep
  repo: https://github.com/SciML/OrdinaryDiffEq.jl
- name: SymBoltz
  role: Supplies the stiff ΛCDM Einstein-Boltzmann problem compiled with an analytical
    sparse Jacobian (CosmologyProblem(jac=true, sparse=true)) and its background subsystem
    prob.bg
  repo: https://github.com/hersle/SymBoltz.jl
- name: Plots
  role: Renders the log-log work-precision diagram of solve time vs L2 error, one
    curve per solver
- name: BenchmarkTools
  role: Timing of repeated runs; timing is recorded as the minimum over N=5 repeated
    runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:e2edafc11ae40a13cf636bf36582f8f783bb612694f9ee00bb5bf5edcf299bfb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SciML/OrdinaryDiffEq.jl
  tools:
  - BenchmarkTools
  - Julia
  - OrdinaryDiffEq.jl
  - Plots
  - SymBoltz
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# Stiff ODE solver work-precision benchmarking

## Summary

Systematically compare stiff implicit ODE solvers (Rodas4/4P/5/5P/6P, FBDF, QNDF) by sweeping shared abstol=reltol tolerances and plotting solve time versus L2 error against a tight-tolerance reference, yielding a log-log work-precision diagram that identifies the best solver and tolerance for a stiff problem such as SymBoltz's Einstein-Boltzmann equations.

## When to use

Use when a stiff ODE system must be integrated with an implicit OrdinaryDiffEq.jl solver and you need to choose a solver and tolerance based on quantitative time-vs-accuracy tradeoffs rather than defaults or intuition.

## When NOT to use

- The problem is non-stiff and explicit solvers (e.g. Tsit5) would be appropriate; the swept Rodas/FBDF/QNDF implicit solvers target stiff systems like the Einstein-Boltzmann equations.
- The goal is absolute cross-code accuracy comparison against an external solver (e.g. CLASS spectra agreement within 0.1%) rather than internal solver selection.
- A single solver/tolerance has already been validated and only one run is needed; the 5-repeat minimum-timing protocol is only meaningful for benchmarking.

## Inputs

- Compiled stiff ODE problem (e.g. SymBoltz CosmologyProblem with jac=true, sparse=true), background subsystem prob.bg
- Solver list with factorization choice (Rodas4, Rodas5, Rodas4P, Rodas5P, Rodas6P, FBDF, QNDF with RFLUFactorization)
- Tolerance sweep grid: abstol=reltol ∈ {1/10^7, ..., 1/10^11}
- Reference solver settings (Rodas5P, abstol=reltol=1e-12)

## Outputs

- Table of (solver, tolerance) pairs with minimum-over-5-runs wall time and L2 error norm(u-ref)/length(u), NaN/NaN for failed retcodes
- Log-log work-precision diagram (Plots.jl), one time-vs-error curve per solver with distinct markers
- Record of tolerances at which FBDF and QNDF produce failed integrations (instability record)

## How to apply

First generate a trusted reference solution: for SymBoltz, compile the ΛCDM model with CosmologyProblem(jac=true, sparse=true) for an analytical sparse Jacobian, solve only the background system (prob.bg) with Rodas5P using RFLUFactorization at abstol=reltol=1e-12. Then sweep abstol=reltol over 1/10^7 through 1/10^11 for each candidate solver (Rodas4, Rodas5, Rodas4P, Rodas5P, Rodas6P, FBDF, QNDF, all with RFLUFactorization), solving prob.bg at each setting. For each solver/tolerance pair, interpolate the solution onto the reference time grid, compute L2 error as norm(u-ref)/length(u), and record the minimum wall time over N=5 repeated runs; mark unsuccessful retcodes as (NaN, NaN) instability points. Plot time-vs-error curves per solver in log-log axes with Plots; rationale is that efficient high-order implicit methods, fast generated code, and sparse Jacobian handling let the full stiff equations be solved without approximation switching at performance comparable to approximation-based codes.

## Related tools

- **OrdinaryDiffEq.jl** (Provides the stiff implicit solvers (Rodas4/4P/5/5P/6P, FBDF, QNDF) and RFLUFactorization used in the tolerance sweep) — https://github.com/SciML/OrdinaryDiffEq.jl
- **SymBoltz** (Supplies the stiff ΛCDM Einstein-Boltzmann problem compiled with an analytical sparse Jacobian (CosmologyProblem(jac=true, sparse=true)) and its background subsystem prob.bg) — https://github.com/hersle/SymBoltz.jl
- **Plots** (Renders the log-log work-precision diagram of solve time vs L2 error, one curve per solver)
- **BenchmarkTools** (Timing of repeated runs; timing is recorded as the minimum over N=5 repeated runs)

## Examples

```
prob = CosmologyProblem(M, p; jac = true, sparse = true); ref = solve(prob.bg, Rodas5P(; linsolve = RFLUFactorization()), abstol = 1e-12, reltol = 1e-12)
```

## Evaluation signals

- For stable solvers, L2 error decreases monotonically toward the 1e-12 reference as tolerances tighten across 1/10^7 → 1/10^11.
- FBDF and QNDF exhibit failed retcodes (recorded as NaN, NaN points) at several of the swept tolerances, as reported in the paper.
- Each (solver, tolerance) timing equals the minimum over N=5 repeated runs, not a single run.
- Error is computed on the reference time grid via interpolation, as norm(u-ref)/length(u).
- The resulting diagram should support the paper's claim of performance comparable to approximation-based codes.

## Limitations

- FBDF and QNDF fail (unsuccessful retcodes) at several of the swept tolerances, so they are not uniformly usable across the tolerance range.
- Timing results are machine- and BLAS-dependent; Julia should be run with multiple threads, and BLAS choice (MKL.jl, AppleAccelerate.jl) can affect performance.
- The L2 error metric norm(u-ref)/length(u) is defined against a Rodas5P 1e-12 reference and inherits any error in that reference solution.

## Evidence

- [abstract] Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods.: "Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods."
- [methods] work-precision diagram: "This plot compares the time to solve the background vs. accuracy of the solution using different ODE solvers and tolerances."
- [methods] analytical sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [abstract] Implicit solvers integrate the full stiff equations at all times without approximations.: "Implicit solvers integrate the full stiff equations at all times without approximations, which greatly simplifies the code."
- [methods] multithreading: "SymBoltz parallelizes integration of different perturbation $k$-modes with multithreading by default."
