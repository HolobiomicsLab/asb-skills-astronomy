---
name: reverse-mode-automatic-differentiation
description: Use when a loss depends on the solution of a neural ODE or neural CDE
  solve in Diffrax/JAX and gradients w.r.t. y0 or parameters are needed, especially
  when long time series would exhaust memory with discretise-then-optimise.
when_to_use_negative:
- 'Memory is not a constraint and the solve has few steps: backpropagating through
  the solver internals (discretise-then-optimise with RecursiveCheckpointAdjoint)
  differentiates the exact discretisation actually used, with no adjoint-ODE approximation
  error'
- The vector field or control interpolation is not smooth enough for the adjoint ODE
  derivation, e.g. unmeasurable controls where even rectilinear interpolation properties
  are insufficient
- The task is training rather than verification of the adjoint method itself — use
  the library's built-in adjoint handling instead of hand-deriving a reverse-mode
  solver
tools:
- name: Diffrax
  role: 'Provides the ODE/CDE solvers and the two adjoint strategies compared: RecursiveCheckpointAdjoint
    (discretise-then-optimise) and BacksolveAdjoint (optimise-then-discretise, implemented
    as a reverse-mode solver)'
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Autodiff framework underlying Diffrax; must be configured to use 64-bit precision
    instead of the 32-bit default for the gradient comparison
  repo: http://github.com/google/jax
- name: torchdiffeq
  role: PyTorch counterpart implementing the adjoint method for ODE backpropagation
    with constant memory cost
  repo: https://github.com/rtqichen/torchdiffeq
- name: torchcde
  role: Prototype PyTorch library for CDEs supporting backpropagation through the
    solver or via the adjoint method
  repo: https://github.com/patrick-kidger/torchcde
- name: pytest
  role: Runs the assertion that optimise-then-discretise and discretise-then-optimise
    gradients agree within discretisation tolerance (via 'uv run pytest')
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:72aebaf7309df2ad0ad50a783e7b1f4a71423bdc05b3a51437ca9adb852e0708
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
  - torchcde
  - torchdiffeq
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# reverse-mode-automatic-differentiation

## Summary

Backpropagates gradients through a numerical ODE/CDE solve either by autodiff through the solver internals (discretise-then-optimise, e.g. Diffrax's RecursiveCheckpointAdjoint) or via a continuous adjoint solved backwards in time (optimise-then-discretise, e.g. Diffrax's BacksolveAdjoint), which is required for constant-memory gradients on long time series.

## When to use

Use when a loss depends on the solution of a neural ODE or neural CDE solve in Diffrax/JAX and gradients w.r.t. y0 or parameters are needed, especially when long time series would exhaust memory with discretise-then-optimise.

## When NOT to use

- Memory is not a constraint and the solve has few steps: backpropagating through the solver internals (discretise-then-optimise with RecursiveCheckpointAdjoint) differentiates the exact discretisation actually used, with no adjoint-ODE approximation error
- The vector field or control interpolation is not smooth enough for the adjoint ODE derivation, e.g. unmeasurable controls where even rectilinear interpolation properties are insufficient
- The task is training rather than verification of the adjoint method itself — use the library's built-in adjoint handling instead of hand-deriving a reverse-mode solver

## Inputs

- Neural ODE vector field dy/dt = f(t, y(t)) (Equinox/Diffrax model) with fixed initial condition and integration interval [t0, t1]
- Reduced neural CDE with the control taken as an interpolation of discrete data (reduced to ODE form)
- Loss function defined on the terminal state y(t1)

## Outputs

- Gradients d(y_N)/d(y_0) and parameter gradients for the vector field parameters
- Per-parameter relative differences between optimise-then-discretise and discretise-then-optimise gradients in adjoint_gradient_comparison.csv
- A pytest test asserting the two gradient estimates agree within discretisation tolerance

## How to apply

First compute reference gradients by backpropagating through the internals of the solver using Diffrax's RecursiveCheckpointAdjoint (discretise-then-optimise), on a fixed small neural ODE dy/dt = f(t, y(t)) with deterministic vector field, fixed y0 and interval [t0, t1]. Then implement the optimise-then-discretise route: analytically derive the backwards-in-time adjoint ODE (for CDEs, construct a backwards-in-time CDE, reduced to an ODE form when the control is an interpolation of discrete data) and numerically solve it in the style of diffrax.BacksolveAdjoint. Enable 64-bit precision instead of JAX's 32-bit default, since adjoint gradients are sensitive to solver tolerances. Repeat the comparison on a reduced neural CDE and compute relative differences between the two gradient estimates per parameter, recording them in adjoint_gradient_comparison.csv.

## Related tools

- **Diffrax** (Provides the ODE/CDE solvers and the two adjoint strategies compared: RecursiveCheckpointAdjoint (discretise-then-optimise) and BacksolveAdjoint (optimise-then-discretise, implemented as a reverse-mode solver)) — https://github.com/patrick-kidger/diffrax
- **JAX** (Autodiff framework underlying Diffrax; must be configured to use 64-bit precision instead of the 32-bit default for the gradient comparison) — http://github.com/google/jax
- **torchdiffeq** (PyTorch counterpart implementing the adjoint method for ODE backpropagation with constant memory cost) — https://github.com/rtqichen/torchdiffeq
- **torchcde** (Prototype PyTorch library for CDEs supporting backpropagation through the solver or via the adjoint method) — https://github.com/patrick-kidger/torchcde
- **pytest** (Runs the assertion that optimise-then-discretise and discretise-then-optimise gradients agree within discretisation tolerance (via 'uv run pytest'))

## Examples

```
import diffrax; sol = diffrax.diffeqsolve(diffrax.ODETerm(f), diffrax.Tsit5(), t0, t1, dt0, y0, adjoint=diffrax.BacksolveAdjoint())
```

## Evaluation signals

- Relative differences between BacksolveAdjoint (optimise-then-discretise) and RecursiveCheckpointAdjoint (discretise-then-optimise) gradients fall within discretisation tolerance for every parameter
- adjoint_gradient_comparison.csv contains a per-parameter record of the relative gradient differences
- 'uv run pytest' passes including the added test asserting the two gradient estimates agree
- Gradients are computed in float64 (64-bit JAX precision) rather than the 32-bit default
- The adjoint solve reproduces the same terminal loss gradient when integrated backwards from the same terminal state and cotangent

## Limitations

- Optimise-then-discretise backpropagation was needed in this work simply to avoid running out of memory on long time series — it introduces an approximation relative to differentiating the actual discretisation
- For CDEs the adjoint requires constructing a backwards-in-time CDE, which must then be numerically solved by reduction to an ODE or otherwise
- No software version numbers are stated for any tool (Diffrax/JAX), so reproducing exact gradient agreement may require pinning versions
- Measurability of the interpolated control limits which adjoint formulations are valid (Hermite cubic splines with backward differences recommended; rectilinear interpolation when measurability is insufficient)

## Evidence

- [readme] torchdiffeq adjoint method: [evidence span withheld — anchor status: not_found]
- [readme] torchcde adjoint support: [evidence span withheld — anchor status: not_found]
- [discussion] Reference implementation available in Diffrax: [evidence span withheld — anchor status: not_found]
