---
name: cosmological-boltzmann-solver-benchmarking
description: Use when measuring wall-clock performance or thread scaling of a cosmological
  Boltzmann solver run (e.g. SymBoltz integrating many k-modes), or when validating
  that an approximation-free implementation achieves performance comparable to approximation-based
  codes.
when_to_use_negative:
- When the goal is accuracy validation of output spectra against CLASS (0.1% agreement)
  rather than timing — use a work-precision diagram instead
- When benchmarking scalar parameter derivatives/autodiff cost rather than k-mode
  integration throughput
- When Julia is running single-threaded only, since the thread=true/false contrast
  cannot produce a meaningful speedup measurement
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3360
tools:
- name: SymBoltz
  role: Cosmology solver whose perturbation k-mode integration is parallelized with
    multithreading and being benchmarked
  repo: github.com/hersle/SymBoltz.jl
- name: BenchmarkTools
  role: BenchmarkGroup timing of solve(prob, ks; thread=true/false) with samples=5,
    seconds=30
- name: Plots
  role: Plotting the timing comparison across thread configurations
- name: OrdinaryDiffEq.jl
  role: Underlying implicit ODE solver machinery used by SymBoltz for the stiff Einstein-Boltzmann
    equations
  repo: github.com/SciML/OrdinaryDiffEq.jl
- name: CLASS
  role: Established approximation-based baseline code for performance comparison
  repo: github.com/lesgourg/class_public
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:e7065a79b5643a7200883b6bd602dc085903323388222d936058725e5a83045b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BenchmarkTools
  - CLASS
  - Julia
  - OrdinaryDiffEq.jl
  - Plots
  - SymBoltz
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cosmological-boltzmann-solver-benchmarking

## Summary

Benchmark and quantify the multithreading speedup of SymBoltz's parallel integration of perturbation k-modes by timing solve(prob, ks) with thread=true vs thread=false using BenchmarkTools, to show the approximation-free solver stays comparable to approximation-based codes.

## When to use

Use when measuring wall-clock performance or thread scaling of a cosmological Boltzmann solver run (e.g. SymBoltz integrating many k-modes), or when validating that an approximation-free implementation achieves performance comparable to approximation-based codes.

## When NOT to use

- When the goal is accuracy validation of output spectra against CLASS (0.1% agreement) rather than timing — use a work-precision diagram instead
- When benchmarking scalar parameter derivatives/autodiff cost rather than k-mode integration throughput
- When Julia is running single-threaded only, since the thread=true/false contrast cannot produce a meaningful speedup measurement

## Inputs

- SymBoltz ΛCDM model instance with parameters_Planck18 parameter values
- CosmologyProblem compiled with jac=true, sparse=true (analytical sparse Jacobian)
- Wavenumber vector ks (e.g. 50 log-spaced modes from 0.1 to 10000 in H0/c units)
- Julia session started with multiple threads enabled

## Outputs

- BenchmarkTools BenchmarkGroup results with minimum and mean wall-clock times per thread configuration
- Timing comparison plot (Plots, e.g. 800x400) of 1-thread vs multi-thread runs
- Table/summary of multithreading speedup (single-thread time / multi-thread time)

## How to apply

Instantiate the minimal ΛCDM model in SymBoltz, load parameters with parameters_Planck18, and compile the problem with CosmologyProblem(jac=true, sparse=true) so an analytical sparse Jacobian is used during integration. Define a wavenumber sweep ks = 10 .^ range(-1, 4, length = 50) covering k from 0.1 to 10000 in H0/c units. Build a BenchmarkTools BenchmarkGroup with two labeled cases — solve(prob, ks; thread = true) on all available Julia threads and solve(prob, ks; thread = false) on a single thread — benchmarked with samples=5, seconds=30. Run the group with BenchmarkTools.run(verbose), record minimum/mean times per configuration, and ensure Julia is actually launched with multiple threads. Plot timings with Plots (e.g. size 800x400) and compute the speedup as single-thread time divided by multi-thread time; the rationale is that SymBoltz parallelizes independent stiff k-mode integrations (solved by implicit ODE methods with sparse Jacobians), so thread scaling is the key performance lever.

## Related tools

- **SymBoltz** (Cosmology solver whose perturbation k-mode integration is parallelized with multithreading and being benchmarked) — github.com/hersle/SymBoltz.jl
- **BenchmarkTools** (BenchmarkGroup timing of solve(prob, ks; thread=true/false) with samples=5, seconds=30)
- **Plots** (Plotting the timing comparison across thread configurations)
- **OrdinaryDiffEq.jl** (Underlying implicit ODE solver machinery used by SymBoltz for the stiff Einstein-Boltzmann equations) — github.com/SciML/OrdinaryDiffEq.jl
- **CLASS** (Established approximation-based baseline code for performance comparison) — github.com/lesgourg/class_public

## Examples

```
using SymBoltz, BenchmarkTools; prob = CosmologyProblem(M, parameters_Planck18(M); jac=true, sparse=true); ks = 10 .^ range(-1, 4, length=50); @btime solve($prob, $ks; thread=true)
```

## Evaluation signals

- All 50 k-modes solve with a successful solver return code in both thread=true and thread=false configurations
- The plot/table reports measured wall-clock times for both '1 thread' and '<nthreads()> threads' cases
- Speedup is computed as single-thread time divided by multi-thread time and is > 1 for multi-threaded execution
- Benchmark timings use BenchmarkTools with samples=5, seconds=30 per case
- Problem was compiled with jac=true, sparse=true so timings reflect the analytical sparse Jacobian configuration

## Limitations

- Timings depend on Julia being launched with multiple threads; single-threaded sessions cannot demonstrate the speedup
- BLAS backend choice (OpenBLAS vs MKL.jl/AppleAccelerate.jl) can affect performance and should be kept consistent between runs
- Benchmark samples=5/seconds=30 gives statistical noise; minimum times are the most stable statistic
- SymBoltz still lacks fast reverse-mode automatic differentiation of scalar loss functions and some additional features

## Evidence

- [methods] SymBoltz parallelizes integration of different perturbation $k$-modes with multithreading by default.: "SymBoltz parallelizes integration of different perturbation $k$-modes with multithreading by default."
- [abstract] Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods.: "Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods."
- [methods] prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [methods] Make sure you [run Julia with multiple threads](https://docs.julialang.org/en/v1/manual/multi-threading/).: "Make sure you [run Julia with multiple threads](https://docs.julialang.org/en/v1/manual/multi-threading/)."
- [intro] Full equations are solved at all times without tight-coupling, ultrarelativistic fluid and radiation-streaming approximations (TCA, UFA and RSA) using efficient implicit ODE: "Full equations are solved at all times without tight-coupling, ultrarelativistic fluid and radiation-streaming approximations (TCA, UFA and RSA) using efficient implicit ODE"
