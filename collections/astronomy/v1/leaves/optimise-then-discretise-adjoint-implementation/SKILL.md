---
name: optimise-then-discretise-adjoint-implementation
description: Use when gradients must be backpropagated through a neural ODE or neural
  CDE solve with constant memory cost, or when comparing optimise-then-discretise
  vs discretise-then-optimise gradient agreement on long time series.
when_to_use_negative:
- The input is already handled fine by discretise-then-optimise backpropagation through
  solver internals (e.g. RecursiveCheckpointAdjoint) and memory is not a constraint
  — the adjoint adds derivation and implementation complexity for no benefit
- The solve is a Stratonovich SDE rather than an ODE/CDE — the thesis derives a separate
  optimise-then-discretise result for SDEs via rough differential equations (Theorem
  C.37), so this ODE/CDE adjoint does not directly apply
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Diffrax
  role: Provides the numerical ODE/CDE solvers, RecursiveCheckpointAdjoint reference,
    and the BacksolveAdjoint style in which the continuous adjoint reverse-mode solver
    is implemented
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Framework in which diffrax is written; autodifferentiation is performed here,
    with 64-bit precision enabled instead of the 32-bit default
  repo: http://github.com/google/jax
- name: pytest
  role: Runs the test asserting optimise-then-discretise and discretise-then-optimise
    gradients agree within discretisation tolerance, via 'uv run pytest'
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:3f7b24e2340e734f884a94a288f840327ef946c8c7f91e7edc071488d9d9c2c7
  source_papers:
  - doi: 10.48550/arxiv.2202.02435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/patrick-kidger/diffrax
  tools:
  - Diffrax
  - JAX
  - pytest
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# optimise-then-discretise-adjoint-implementation

## Summary

Implements continuous adjoint (optimise-then-discretise) backpropagation through neural ODE/CDE solves by analytically deriving a backwards-in-time adjoint ODE and numerically solving it, in the style of diffrax.BacksolveAdjoint, then validating against discretise-then-optimise gradients.

## When to use

Use when gradients must be backpropagated through a neural ODE or neural CDE solve with constant memory cost, or when comparing optimise-then-discretise vs discretise-then-optimise gradient agreement on long time series.

## When NOT to use

- The input is already handled fine by discretise-then-optimise backpropagation through solver internals (e.g. RecursiveCheckpointAdjoint) and memory is not a constraint — the adjoint adds derivation and implementation complexity for no benefit
- The solve is a Stratonovich SDE rather than an ODE/CDE — the thesis derives a separate optimise-then-discretise result for SDEs via rough differential equations (Theorem C.37), so this ODE/CDE adjoint does not directly apply

## Inputs

- Neural ODE dy/dt = f(t, y(t)) with deterministic vector field, fixed initial condition, and interval [t0, t1]
- Reduced neural CDE with control taken as an interpolation of discrete data, reduced to an ODE form
- Diffrax solver internals (e.g. RecursiveCheckpointAdjoint reference gradients)
- Analytically derived backwards-in-time adjoint ODE per Theorem 5.9

## Outputs

- Optimise-then-discretise gradients dy_N/dy_0 and parameter gradients from the adjoint solve
- adjoint_gradient_comparison.csv with relative differences per parameter
- Test asserting gradients agree within discretisation tolerance, passing under 'uv run pytest'

## How to apply

Clone the diffrax repository, install it with dependencies, and enable 64-bit precision in JAX rather than the 32-bit default. Define a fixed small neural ODE dy/dt = f(t, y(t)) with a deterministic vector field, fixed initial condition, and integration interval [t0, t1]; compute reference gradients dy_N/dy_0 and parameter gradients by backpropagating through the solver internals with diffrax's RecursiveCheckpointAdjoint (discretise-then-optimise). Then implement the continuous adjoint as a reverse-mode solver in the style of diffrax.BacksolveAdjoint: analytically derive the backwards-in-time adjoint ODE (per Theorem 5.9) and numerically solve it to obtain gradients (optimise-then-discretise). Repeat the comparison on a reduced neural CDE whose control is an interpolation of discrete data, reduced to an ODE form. Compute relative differences per parameter, record them in adjoint_gradient_comparison.csv, and add a test asserting gradients agree within discretisation tolerance; verify with 'uv run pytest'.

## Related tools

- **Diffrax** (Provides the numerical ODE/CDE solvers, RecursiveCheckpointAdjoint reference, and the BacksolveAdjoint style in which the continuous adjoint reverse-mode solver is implemented) — https://github.com/patrick-kidger/diffrax
- **JAX** (Framework in which diffrax is written; autodifferentiation is performed here, with 64-bit precision enabled instead of the 32-bit default) — http://github.com/google/jax
- **pytest** (Runs the test asserting optimise-then-discretise and discretise-then-optimise gradients agree within discretisation tolerance, via 'uv run pytest')

## Examples

```
import diffrax; adjoint = diffrax.BacksolveAdjoint(); sol = diffrax.diffeqsolve(term, solver=diffrax.Tsit5(), t0=t0, t1=t1, dt0=None, y0=y0, adjoint=adjoint)
```

## Evaluation signals

- Relative differences between optimise-then-discretise and discretise-then-optimise gradients, recorded per parameter in adjoint_gradient_comparison.csv, fall within discretisation tolerance
- The tolerance test asserting gradient agreement passes under 'uv run pytest' with all tests green
- Adjoint gradients dy_N/dy_0 match the RecursiveCheckpointAdjoint reference backpropagated through the solver internals
- The reduced neural CDE (control interpolated from discrete data, reduced to ODE form) yields consistent adjoint gradients with the same comparison workflow

## Limitations

- Optimise-then-discretise backpropagation was needed simply to avoid running out of memory on long time series; for short series discretise-then-optimise may be simpler and sufficient
- The continuous adjoint's accuracy is bounded by the discretisation of the backwards-in-time adjoint ODE, so gradients agree with discretise-then-optimise only within discretisation tolerance, not exactly
- Software version numbers are not stated for any tool in the article's methods

## Evidence

- [readme] adjoint method support in solvers: [evidence span withheld — anchor status: not_found]
