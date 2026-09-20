---
name: background-cosmology-integration
description: Use when benchmarking stiff ODE integration of a cosmological (or similarly
  stiff physical) model subsystem against a low-tolerance reference solution, especially
  to quantify solver stability and accuracy trade-offs across tolerance sweeps.
when_to_use_negative:
- The goal is validating output spectra (matter or CMB power spectra) against CLASS
  rather than benchmarking solver time-error behavior
- The model is non-stiff or the subsystem cannot be solved implicitly, so explicit
  solvers would be appropriate and the RFLUFactorization/Rodas setup is unnecessary
- Fast reverse-mode automatic differentiation of scalar loss functions is required
  — the paper notes this is not yet implemented
edam_operation: http://edamontology.org/operation_2426
tools:
- name: SymBoltz
  role: Provides the symbolic-numeric ΛCDM model, parameters_Planck18, and compiled
    CosmologyProblem whose background subsystem (prob.bg) is benchmarked
  repo: https://github.com/hersle/SymBoltz.jl
- name: OrdinaryDiffEq.jl
  role: Supplies the implicit stiff solvers (Rodas4/5/4P/5P/6P, FBDF, QNDF) and RFLUFactorization
    linear solver used in the tolerance sweep
  repo: https://github.com/SciML/OrdinaryDiffEq.jl
- name: Plots
  role: Used to render the log-log time-vs-error work-precision diagram with one curve
    and distinct markers per solver
- name: BenchmarkTools
  role: Timing of repeated solver runs; the recorded timing is the minimum over N=5
    repeats
- name: CLASS
  role: Established approximation-based code serving as the performance comparison
    baseline for implicit integration
  repo: https://github.com/lesgourg/class_public
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202557450
    title: 'SymBoltz.jl: A symbolic-numeric, approximation-free, and differentiable
      linear Einstein–Boltzmann solver'
  source_content_hash: sha256:2e956c3c2e343d781ea0a2a6005e10b3f993bf7abe8b9bb4ec1ffd7db2b107a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hersle/SymBoltz.jl
  tools:
  - BenchmarkTools
  - CLASS
  - Julia
  - OrdinaryDiffEq.jl
  - Plots
  - SymBoltz
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# background-cosmology-integration

## Summary

Benchmark implicit ODE solvers on the stiff background subsystem of a symbolic-numeric Einstein-Boltzmann model by sweeping tolerances and building a work-precision diagram. It demonstrates that SymBoltz's approximation-free integration performs comparably to approximation-based codes like CLASS.

## When to use

Use when benchmarking stiff ODE integration of a cosmological (or similarly stiff physical) model subsystem against a low-tolerance reference solution, especially to quantify solver stability and accuracy trade-offs across tolerance sweeps.

## When NOT to use

- The goal is validating output spectra (matter or CMB power spectra) against CLASS rather than benchmarking solver time-error behavior
- The model is non-stiff or the subsystem cannot be solved implicitly, so explicit solvers would be appropriate and the RFLUFactorization/Rodas setup is unnecessary
- Fast reverse-mode automatic differentiation of scalar loss functions is required — the paper notes this is not yet implemented

## Inputs

- SymBoltz ΛCDM symbolic cosmology model with Planck18 parameter values (parameters_Planck18)
- Compiled CosmologyProblem with analytical sparse Jacobian (jac=true, sparse=true)
- Background subsystem problem prob.bg

## Outputs

- Per solver/tolerance L2 errors (norm(u-ref)/length(u)) against the 1e-12 Rodas5P reference
- Minimum wall-clock timings over N=5 repeated runs, with (NaN, NaN) markers for failed retcodes
- Log-log work-precision diagram (time vs error) with one curve per solver
- Instability record listing tolerances at which FBDF and QNDF fail

## How to apply

Build the SymBoltz ΛCDM model, load Planck18 parameter values via parameters_Planck18, and compile with CosmologyProblem(jac=true, sparse=true) to obtain an analytical sparse Jacobian. First generate a reference solution by solving only the background system (prob.bg) with Rodas5P using RFLUFactorization at abstol=reltol=1e-12. Then sweep abstol=reltol from 1/10^7 to 1/10^11 for Rodas4, Rodas5, Rodas4P, Rodas5P, Rodas6P, FBDF and QNDF (all with RFLUFactorization); interpolate each solution onto the reference time grid, compute the L2 error as norm(u-ref)/length(u), and record timing as the minimum over N=5 repeated runs, marking failed retcodes as (NaN, NaN) instability points. Plot the time-vs-error diagram in log-log axes with Plots, one curve per solver; rationale is to show that high-order implicit methods with sparse Jacobians keep error decreasing monotonically toward the 1e-12 reference while remaining performant.

## Related tools

- **SymBoltz** (Provides the symbolic-numeric ΛCDM model, parameters_Planck18, and compiled CosmologyProblem whose background subsystem (prob.bg) is benchmarked) — https://github.com/hersle/SymBoltz.jl
- **OrdinaryDiffEq.jl** (Supplies the implicit stiff solvers (Rodas4/5/4P/5P/6P, FBDF, QNDF) and RFLUFactorization linear solver used in the tolerance sweep) — https://github.com/SciML/OrdinaryDiffEq.jl
- **Plots** (Used to render the log-log time-vs-error work-precision diagram with one curve and distinct markers per solver)
- **BenchmarkTools** (Timing of repeated solver runs; the recorded timing is the minimum over N=5 repeats)
- **CLASS** (Established approximation-based code serving as the performance comparison baseline for implicit integration) — https://github.com/lesgourg/class_public

## Examples

```
prob = CosmologyProblem(M, p; jac = true, sparse = true); ref = solve(prob.bg, Rodas5P(linsolve = RFLUFactorization()), abstol = 1e-12, reltol = 1e-12)
```

## Evaluation signals

- For stable solvers (Rodas family), the L2 error decreases monotonically toward the 1e-12 reference as tolerances tighten
- FBDF and QNDF exhibit failed integrations (non-Success retcodes mapped to (NaN, NaN)) at several swept tolerances, as noted in the paper
- Timings are the minimum over N=5 repeated runs, reducing jitter in the work-precision diagram
- The compiled problem uses the analytical sparse Jacobian (jac=true, sparse=true), which the paper credits for performance comparable to approximation-based codes
- Background solution at the tightest tolerance matches the Rodas5P RFLUFactorization reference on the interpolated time grid

## Limitations

- FBDF and QNDF are unstable for this stiff background system and fail (bad retcodes) at several of the swept tolerances (1/10^7 through 1/10^11)
- Fast reverse-mode automatic differentiation of scalar loss functions is not yet implemented; more work is needed on additional features
- Performance conclusions are specific to the background system with RFLUFactorization and may not transfer to the full perturbation integration

## Evidence

- [abstract] Performance comparable to approximation-based codes: "Performance remains comparable to existing approximation-based codes due to efficient high-order implicit methods, fast generated code, optimal handling of the Jacobian, and sparse matrix methods."
- [methods] Compiled problem with analytical sparse Jacobian: "prob = CosmologyProblem(M, p; jac = true, sparse = true) # analytical+sparse Jacobian"
- [methods] Work-precision diagram rationale: "This plot compares the time to solve the background vs. accuracy of the solution using different ODE solvers and tolerances."
- [abstract] Implicit solvers integrate full equations: "Implicit solvers integrate the full stiff equations at all times without approximations, which greatly simplifies the code."
- [readme] SymBoltz package purpose: "SymBoltz.jl is a Julia package for solving the linear Einstein-Boltzmann equations in cosmology."
