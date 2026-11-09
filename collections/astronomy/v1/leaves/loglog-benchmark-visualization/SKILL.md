---
name: loglog-benchmark-visualization
description: Use when you need to compare multiple implicit ODE solvers/tolerances
  by runtime-versus-accuracy, e.g. sweeping abstol=reltol from 1/10^7 to 1/10^11 against
  a fixed low-tolerance reference solution.
when_to_use_negative:
- When only a single solver at a single tolerance is needed — a work-precision sweep
  adds unnecessary repeated benchmarking
- When the quantity of interest is agreement with CLASS spectra (e.g. 0.1% output-spectrum
  accuracy) rather than ODE solver runtime/accuracy trade-off
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SymBoltz
  role: Provides the stiff Einstein-Boltzmann problem (prob.bg) being benchmarked
    and parameter values via parameters_Planck18
  repo: github.com/hersle/SymBoltz.jl
- name: OrdinaryDiffEq.jl
  role: Supplies the implicit solvers (Rodas4/5/6P, Rodas4P/5P, FBDF, QNDF) and RFLUFactorization
    used across the tolerance sweep
  repo: github.com/SciML/OrdinaryDiffEq.jl
- name: Plots
  role: Renders the log-log time-vs-error work-precision diagram with one marked curve
    per solver
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:c94f676f2e698d92b8179e746ac4cfdd4ab79e61f4b3b885174732deff76f48f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia
  - OrdinaryDiffEq.jl
  - Plots
  - SymBoltz
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# loglog-benchmark-visualization

## Summary

Constructs a work-precision diagram plotting solve time against L2 error on log-log axes to compare ODE solvers (e.g. Rodas4/5/6P, FBDF, QNDF) across tolerance sweeps on SymBoltz background problems.

## When to use

Use when you need to compare multiple implicit ODE solvers/tolerances by runtime-versus-accuracy, e.g. sweeping abstol=reltol from 1/10^7 to 1/10^11 against a fixed low-tolerance reference solution.

## When NOT to use

- When only a single solver at a single tolerance is needed — a work-precision sweep adds unnecessary repeated benchmarking
- When the quantity of interest is agreement with CLASS spectra (e.g. 0.1% output-spectrum accuracy) rather than ODE solver runtime/accuracy trade-off

## Inputs

- Compiled SymBoltz CosmologyProblem with analytical sparse Jacobian (jac=true, sparse=true); specifically the background sub-problem prob.bg
- Reference solution from Rodas5P at abstol=reltol=1e-12 with RFLUFactorization
- Solver/tolerance sweep grid: {Rodas4, Rodas5, Rodas4P, Rodas5P, Rodas6P, FBDF, QNDF} x {1/10^7, ..., 1/10^11}

## Outputs

- A time-vs-error work-precision diagram (log-log Plots figure, one curve per solver with distinct markers)
- Per solver/tolerance records of (minimum timing over N=5 runs, L2 error norm(u-ref)/length(u)), with (NaN, NaN) for failed retcodes

## How to apply

First generate a reference solution by solving prob.bg (the background system of a CosmologyProblem compiled with jac=true, sparse=true) with Rodas5P using RFLUFactorization and abstol=reltol=1e-12. Then sweep abstol=reltol over 1/10^7 through 1/10^11 for Rodas4, Rodas5, Rodas4P, Rodas5P, Rodas6P, FBDF and QNDF (all with RFLUFactorization). For each solver/tolerance pair, interpolate the solution onto the reference time grid, compute the L2 error as norm(u-ref)/length(u), and record the timing as the minimum over N=5 repeated runs to suppress noise. Mark failed retcodes as (NaN, NaN) instability points so they appear as gaps rather than spurious points. Finally, plot time-vs-error in log-log axes with Plots, one curve per solver with distinct markers; rationale is that log-log axes make the trade-off between tolerance, accuracy and cost directly comparable across solvers spanning orders of magnitude.

## Related tools

- **SymBoltz** (Provides the stiff Einstein-Boltzmann problem (prob.bg) being benchmarked and parameter values via parameters_Planck18) — github.com/hersle/SymBoltz.jl
- **OrdinaryDiffEq.jl** (Supplies the implicit solvers (Rodas4/5/6P, Rodas4P/5P, FBDF, QNDF) and RFLUFactorization used across the tolerance sweep) — github.com/SciML/OrdinaryDiffEq.jl
- **Plots** (Renders the log-log time-vs-error work-precision diagram with one marked curve per solver)

## Examples

```
plot(times, errors; xscale=:log10, yscale=:log10, marker=:circle, label="Rodas5P", xlabel="time (s)", ylabel="L2 error")
```

## Evaluation signals

- Error decreases monotonically toward the 1e-12 reference as tolerances tighten for stable solvers (Rodas family)
- FBDF and QNDF exhibit (NaN, NaN) instability points at several swept tolerances, matching the paper's reported failed retcodes
- Recorded timing for each point equals the minimum over N=5 repeated runs
- L2 error values are computed on the reference time grid as norm(u-ref)/length(u)
- Diagram axes are log-log with one distinguishable marker/curve per solver

## Limitations

- FBDF and QNDF fail (non-success retcodes) at several tolerances in the 1/10^7–1/10^11 sweep, leaving gaps in their curves
- Benchmark timings are machine-dependent; minimum-of-5 mitigates but does not eliminate noise, and BLAS choice (e.g. MKL.jl, AppleAccelerate.jl) affects absolute times
- Accuracy is measured relative to a single Rodas5P 1e-12 reference, so the error floor is set by that reference's own accuracy

## Evidence

- [methods] Work-precision diagram purpose: "This plot compares the time to solve the background vs. accuracy of the solution using different ODE solvers and tolerances."
- [methods] Plots usage for figures: "using Plots
p = plot(layout = (3, 1), size = (800, 1000))"
- [methods] Analytical sparse Jacobian compilation: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [abstract] Performance finding attributed to implicit methods: "Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods."
- [readme] SymBoltz package description: "SymBoltz.jl is a Julia package for solving the linear Einstein-Boltzmann equations in cosmology."
