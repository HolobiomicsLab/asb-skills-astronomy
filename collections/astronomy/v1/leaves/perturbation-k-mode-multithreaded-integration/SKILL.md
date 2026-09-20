---
name: perturbation-k-mode-multithreaded-integration
description: Use when reproducing or measuring the multithreading speedup of integrating
  many perturbation k-modes in SymBoltz, or when deciding how many Julia threads to
  allocate for a k-sweep of the linear Einstein-Boltzmann equations.
when_to_use_negative:
- When a single k-mode (not a k-sweep) is being solved — there is no k-mode-level
  parallelism to exploit
- When the target is non-Julia codes such as CLASS, which parallelize differently
  (e.g. OpenMP compilation with -fopenmp) and do not expose a thread=true/false solve
  option
- When the goal is accuracy comparison (work-precision diagrams) rather than wall-clock
  timing; tolerances, not thread counts, are the relevant knobs
edam_operation: http://edamontology.org/operation_3842
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SymBoltz
  role: Provides the approximation-free Einstein-Boltzmann solver whose solve(prob,
    ks; thread=...) API parallelizes k-modes across Julia threads
  repo: https://github.com/hersle/SymBoltz.jl
- name: Julia
  role: Runtime whose multi-threading (run Julia with multiple threads) executes the
    parallel k-mode integration
- name: BenchmarkTools
  role: BenchmarkGroup sampling (samples=5, seconds=30) of the thread=true vs thread=false
    solve calls
- name: Plots
  role: Plots the timing comparison across thread settings (e.g. size 800x400)
- name: OrdinaryDiffEq.jl
  role: Underlying SciML ODE solver infrastructure SymBoltz uses for the implicit
    integration of the stiff equations
  repo: https://github.com/SciML/OrdinaryDiffEq.jl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:498f1b2244fc892578da814010ea71416cbeee30270b2c5f76440f09814d8448
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - BenchmarkTools
  - Julia
  - OrdinaryDiffEq.jl
  - Plots
  - SymBoltz
  edam_operation: http://edamontology.org/operation_3842
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# perturbation-k-mode-multithreaded-integration

## Summary

Benchmark and exploit SymBoltz's default multithreading over perturbation k-modes, comparing 1-thread vs all-Julia-thread wall-clock times of the full stiff Einstein-Boltzmann solve to quantify parallel speedup without approximation switching.

## When to use

Use when reproducing or measuring the multithreading speedup of integrating many perturbation k-modes in SymBoltz, or when deciding how many Julia threads to allocate for a k-sweep of the linear Einstein-Boltzmann equations.

## When NOT to use

- When a single k-mode (not a k-sweep) is being solved — there is no k-mode-level parallelism to exploit
- When the target is non-Julia codes such as CLASS, which parallelize differently (e.g. OpenMP compilation with -fopenmp) and do not expose a thread=true/false solve option
- When the goal is accuracy comparison (work-precision diagrams) rather than wall-clock timing; tolerances, not thread counts, are the relevant knobs

## Inputs

- SymBoltz ΛCDM model with Planck18 parameters (parameters_Planck18)
- Compiled CosmologyProblem with jac=true, sparse=true (analytical sparse Jacobian)
- Wavenumber vector ks = 10 .^ range(-1, 4, length = 50) in H0/c units
- Julia session started with multiple threads (JULIA_NUM_THREADS / --threads)

## Outputs

- BenchmarkTools minimum/mean wall-clock times for '1 thread' and '<nthreads()> threads' cases
- Multithreading speedup values (single-thread time / multi-thread time) per configuration
- Plots timing-comparison figure (e.g. 800x400) and a tabulated speedup summary

## How to apply

Instantiate the ΛCDM model in SymBoltz, set parameters with parameters_Planck18, and compile with CosmologyProblem(jac=true, sparse=true) so the analytical sparse Jacobian is used. Define the wavenumber sweep ks = 10 .^ range(-1, 4, length = 50) (k from 0.1 to 10000 in H0/c units). Build a BenchmarkTools BenchmarkGroup with two labeled cases — solve(prob, ks; thread = true) on all available Julia threads and solve(prob, ks; thread = false) on 1 thread — each benchmarked with samples=5, seconds=30, then run it verbosely and record minimum/mean times per configuration. Compute the speedup as single-thread time / multi-thread time, plot the timing comparison with Plots (e.g. size 800x400), and tabulate it. Rationale: SymBoltz parallelizes integration of different perturbation k-modes with multithreading by default, and its approximation-free implicit solves remain comparable to approximation-based codes like CLASS because of efficient high-order implicit methods, fast generated code, optimal Jacobian handling, and sparse matrix methods.

## Related tools

- **SymBoltz** (Provides the approximation-free Einstein-Boltzmann solver whose solve(prob, ks; thread=...) API parallelizes k-modes across Julia threads) — https://github.com/hersle/SymBoltz.jl
- **Julia** (Runtime whose multi-threading (run Julia with multiple threads) executes the parallel k-mode integration)
- **BenchmarkTools** (BenchmarkGroup sampling (samples=5, seconds=30) of the thread=true vs thread=false solve calls)
- **Plots** (Plots the timing comparison across thread settings (e.g. size 800x400))
- **OrdinaryDiffEq.jl** (Underlying SciML ODE solver infrastructure SymBoltz uses for the implicit integration of the stiff equations) — https://github.com/SciML/OrdinaryDiffEq.jl

## Examples

```
prob = CosmologyProblem(M, parameters_Planck18; jac=true, sparse=true); ks = 10 .^ range(-1, 4, length=50); @btime solve($prob, $ks; thread=true)
```

## Evaluation signals

- Both thread configurations return a successful solver return code for all 50 k-modes
- The reported plot/table contains measured wall-clock times for exactly the '1 thread' and '<nthreads()> threads' cases
- Speedup is computed as single-thread time / multi-thread time and should be > 1 on a multi-threaded Julia session (with diminishing returns beyond physical cores)
- The same problem compilation (jac=true, sparse=true) and identical ks sweep are used in both benchmark cases, so the only differing factor is thread count
- Benchmark statistics are stable (BenchmarkTools samples=5, seconds=30), with minimum times reported alongside means

## Limitations

- Speedup measurement requires that Julia actually be launched with multiple threads; SymBoltz's docs say 'Make sure you run Julia with multiple threads'
- Performance is machine- and BLAS-dependent — MKL.jl (Intel) or AppleAccelerate.jl (Macs) can change linear-algebra timings relative to default OpenBLAS
- The paper notes more work is needed for additional features and fast reverse-mode automatic differentiation of scalar loss functions, so end-to-end differentiable timing claims do not extend there
- The benchmark uses a fixed k-grid (50 log-spaced modes from 0.1 to 10000 H0/c); other grids may shift the parallel-efficiency picture

## Evidence

- [methods] SymBoltz parallelizes integration of different perturbation $k$-modes with multithreading by default.: "SymBoltz parallelizes integration of different perturbation $k$-modes with multithreading by default."
- [methods] analytical+sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [methods] run Julia with multiple threads: "Make sure you [run Julia with multiple threads](https://docs.julialang.org/en/v1/manual/multi-threading/)."
- [abstract] performance comparable via implicit methods and sparse matrices: "Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods."
- [methods] solve for a set of wavenumbers: "ks = [4e0, 4e1, 4e2, 4e3]
sol = solve(prob, ks)"
