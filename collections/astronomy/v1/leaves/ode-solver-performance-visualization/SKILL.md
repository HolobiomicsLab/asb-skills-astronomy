---
name: ode-solver-performance-visualization
description: Use when you need to quantify or visualize the execution time of ODE
  solves (e.g. SymBoltz k-mode sweeps, work-precision trade-offs between solvers/tolerances)
  or measure the speedup from multithreading the integration of independent perturbation
  k-modes.
when_to_use_negative:
- When the goal is accuracy validation of spectra against CLASS (0.1% agreement) rather
  than runtime measurement — use solver-accuracy comparison workflows instead
- When the ODE system is small or non-stiff enough that single-threaded solve time
  is negligible and benchmarking overhead dominates
- When reverse-mode automatic differentiation cost of a scalar loss function is the
  target — the paper notes fast reverse-mode AD is not yet implemented
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SymBoltz
  role: Provides the approximation-free Einstein-Boltzmann solver whose solve(prob,
    ks; thread) call is benchmarked, with built-in multithreading across k-modes
  repo: https://github.com/hersle/SymBoltz.jl
- name: BenchmarkTools
  role: BenchmarkGroup sampling (samples=5, seconds=30) of threaded and unthreaded
    solve timings
- name: Plots
  role: Renders the timing comparison and work-precision plots (e.g. layout (3,1),
    size 800x1000)
- name: OrdinaryDiffEq.jl
  role: Supplies the high-order implicit ODE solvers and tolerances evaluated in the
    work-precision diagram
  repo: https://github.com/SciML/OrdinaryDiffEq.jl
- name: MKL.jl
  role: Optional BLAS/LAPACK backend swap to accelerate the sparse linear algebra
    in implicit solves
  repo: https://github.com/JuliaLinearAlgebra/MKL.jl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:e37b01352e580d12cee17abc6edb5456c0228dfb1269f3e9310b41d0de841959
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - BenchmarkTools
  - Julia
  - MKL.jl
  - OrdinaryDiffEq.jl
  - Plots
  - SymBoltz
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ODE solver performance visualization

## Summary

Benchmark and plot the runtime of SymBoltz's implicit ODE solves of the stiff Einstein-Boltzmann equations, comparing multithreaded vs single-threaded perturbation k-mode integration and solver/tolerance configurations to demonstrate performance comparable to approximation-based codes like CLASS.

## When to use

Use when you need to quantify or visualize the execution time of ODE solves (e.g. SymBoltz k-mode sweeps, work-precision trade-offs between solvers/tolerances) or measure the speedup from multithreading the integration of independent perturbation k-modes.

## When NOT to use

- When the goal is accuracy validation of spectra against CLASS (0.1% agreement) rather than runtime measurement — use solver-accuracy comparison workflows instead
- When the ODE system is small or non-stiff enough that single-threaded solve time is negligible and benchmarking overhead dominates
- When reverse-mode automatic differentiation cost of a scalar loss function is the target — the paper notes fast reverse-mode AD is not yet implemented

## Inputs

- Compiled SymBoltz CosmologyProblem with analytical sparse Jacobian (jac=true, sparse=true)
- ΛCDM parameters set from parameters_Planck18
- Wavenumber vector ks (e.g. 50 log-spaced values from 0.1 to 10000 in H0/c units)
- Julia session launched with multiple threads (JULIA_NUM_THREADS)
- Optional: low-tolerance reference solution for work-precision comparisons

## Outputs

- BenchmarkTools minimum and mean execution times per thread configuration (1 thread vs <nthreads()> threads)
- Plot comparing wall-clock times across thread settings (Plots, e.g. 800x400 px)
- Table of multithreading speedup (single-thread time / multi-thread time) per configuration
- Work-precision diagram of solver time vs accuracy for different ODE solvers and tolerances

## How to apply

Instantiate the ΛCDM model in SymBoltz with parameters_Planck18 and compile it with CosmologyProblem(jac=true, sparse=true) so the analytical sparse Jacobian keeps the implicit solve fast. Define a logarithmic wavenumber sweep such as ks = 10 .^ range(-1, 4, length = 50) (k from 0.1 to 10000 in H0/c units) and build a BenchmarkTools BenchmarkGroup with two labeled cases: solve(prob, ks; thread = true) on all available Julia threads and solve(prob, ks; thread = false) on 1 thread, each benchmarked with samples=5, seconds=30. Run the group with BenchmarkTools.run(verbose=true) and record minimum/mean wall-clock times per configuration; SymBoltz parallelizes integration of different perturbation k-modes with multithreading by default, so running Julia with multiple threads (optionally with MKL.jl or AppleAccelerate.jl for BLAS) is a prerequisite. Plot the timing comparison with Plots (e.g. size 800x400) and tabulate the multithreading speedup (single-thread time / multi-thread time); for solver-accuracy trade-offs, produce a work-precision diagram comparing time to solution versus accuracy of the background against a low-tolerance reference solution.

## Related tools

- **SymBoltz** (Provides the approximation-free Einstein-Boltzmann solver whose solve(prob, ks; thread) call is benchmarked, with built-in multithreading across k-modes) — https://github.com/hersle/SymBoltz.jl
- **BenchmarkTools** (BenchmarkGroup sampling (samples=5, seconds=30) of threaded and unthreaded solve timings)
- **Plots** (Renders the timing comparison and work-precision plots (e.g. layout (3,1), size 800x1000))
- **OrdinaryDiffEq.jl** (Supplies the high-order implicit ODE solvers and tolerances evaluated in the work-precision diagram) — https://github.com/SciML/OrdinaryDiffEq.jl
- **MKL.jl** (Optional BLAS/LAPACK backend swap to accelerate the sparse linear algebra in implicit solves) — https://github.com/JuliaLinearAlgebra/MKL.jl

## Examples

```
prob = CosmologyProblem(M, p; jac=true, sparse=true); ks = 10 .^ range(-1, 4, length = 50); benches = @benchmarkable solve(prob, ks; thread = $thread) samples=5 seconds=30
```

## Evaluation signals

- The solver returns a successful return code for all 50 k-modes in both thread configurations
- Reported wall-clock times exist for both the '1 thread' and '<nthreads()> threads' cases, and the speedup = single-thread time / multi-thread time is computed from them
- Speedup is a positive ratio; near-unity speedup indicates Julia was launched without multiple threads
- Work-precision plot shows solve time increasing monotonically as target accuracy tolerance is tightened, consistent with the paper's background-vs-accuracy comparison
- Benchmark medians/means are stable across BenchmarkTools samples (samples=5, seconds=30), with no compile-time artifacts dominating the first sample

## Limitations

- Measured speedup depends on the host's available Julia threads and BLAS backend; the paper suggests MKL.jl (Intel) or AppleAccelerate.jl (Macs) as alternatives to OpenBLAS
- First benchmark samples include JIT compilation overhead; rely on minimum times or warm-up runs
- Fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented, so performance conclusions do not extend to reverse-mode AD workloads

## Evidence

- [abstract] Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods.: "Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods."
- [methods] SymBoltz parallelizes integration of different perturbation k-modes with multithreading by default.: "SymBoltz parallelizes integration of different perturbation $k$-modes with multithreading by default."
- [methods] This plot compares the time to solve the background vs. accuracy of the solution using different ODE solvers and tolerances.: "This plot compares the time to solve the background vs. accuracy of the solution using different ODE solvers and tolerances."
- [methods] Make sure you [run Julia with multiple threads]: "Make sure you [run Julia with multiple threads]"
