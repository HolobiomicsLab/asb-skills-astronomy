---
name: reversible-heun-step-implementation
description: Use when a Stratonovich neural SDE or reversible ODE solver needs a step
  map whose reverse step reconstructs the initial state to machine precision, or when
  convergence order/stability-region claims must be empirically reproduced.
when_to_use_negative:
- When a high-accuracy non-reversible solver suffices (e.g. Tsit5 or Dopri8 from diffrax)
  and no reversibility constraint exists
- When the noise is not Stratonovich and the additive-noise order-1 guarantee does
  not apply, yet order 1 is required
- When higher-order reversible ODE solvers are needed — the article notes 'it would
  be desirable to have higher order reversible ODE solvers' but they are not yet available
tools:
- name: Diffrax
  role: JAX-based library providing the numerical differential equation solvers (including
    symplectic solvers) in which the ReversibleHeun step maps are implemented and
    exercised
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Backend framework supporting autodifferentiation and vectorised convergence/stability
    sweeps for the step maps
  repo: http://github.com/google/jax
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:c3ac15f79ebc8a33814cd0f07702ce4d7c3ae0585acbbb6d9a698e40ecce229d
  source_papers:
  - doi: 10.48550/arxiv.2202.02435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/patrick-kidger/diffrax
  tools:
  - Diffrax
  - JAX
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# Reversible Heun step implementation

## Summary

Implement and validate the reversible Heun method as matched forward/reverse step maps (Algorithms 2-3), verifying exact reversibility, second-order ODE convergence, strong order 1/2 for Stratonovich SDEs (order 1 under additive noise), and a stability region equal to the complex interval [-i, i] on the Dahlquist test equation, per Theorems 5.18-5.20 (proofs in Appendix C.4).

## When to use

Use when a Stratonovich neural SDE or reversible ODE solver needs a step map whose reverse step reconstructs the initial state to machine precision, or when convergence order/stability-region claims must be empirically reproduced.

## When NOT to use

- When a high-accuracy non-reversible solver suffices (e.g. Tsit5 or Dopri8 from diffrax) and no reversibility constraint exists
- When the noise is not Stratonovich and the additive-noise order-1 guarantee does not apply, yet order 1 is required
- When higher-order reversible ODE solvers are needed — the article notes 'it would be desirable to have higher order reversible ODE solvers' but they are not yet available

## Inputs

- Scalar or vector ODE test problem with known solution (for the convergence study)
- Stratonovich SDE definitions, driven by a VirtualBrownianTree (general and additive-noise variants)
- Complex lambda grid over the plane for the Dahlquist test equation dy/dt = lambda*y
- Terminal solver state (y1 plus final solver state) for the reverse-step reversibility check

## Outputs

- reversible_heun_validation.csv with measured ODE order, SDE strong orders, reversibility errors, and stability-region bounds
- Fitted log-log convergence slopes (expected ~2 for ODEs, ~0.5 general / ~1.0 additive for SDEs)
- Extracted complex-plane stability boundary (expected equal to [-i, i])
- Reversibility reconstruction error (expected at machine/zero-truncation precision)

## How to apply

1) Install the diffrax library on the JAX backend (github.com/patrick-kidger/diffrax). 2) Implement the forward and reverse ReversibleHeun step maps (Algorithms 2-3) and check reversibility: given terminal state y1 plus final solver state, reconstruct y0 and confirm the reconstruction error is at floating-point (zero truncation) precision. 3) Run an ODE convergence study over a step-size refinement grid with ReversibleHeun, fit a log-log regression of error vs step size, and expect a fitted order of ~2 (the two-step update coincides with the leapfrog/midpoint method). 4) Run SDE convergence studies on Stratonovich SDEs driven by a VirtualBrownianTree with unit step size, measuring strong order ~0.5 for general noise and ~1.0 for additive noise. 5) Sweep the complex lambda plane for the Dahlquist test equation dy/dt = lambda*y with unit-step ReversibleHeun and extract the stability boundary, which must equal the segment [-i, i] on the imaginary axis. 6) Write measured orders, reversibility reconstruction errors, and stability-region bounds to reversible_heun_validation.csv; accept only if all four criteria pass (ODE order ~2, SDE orders ~0.5/1.0, reversibility at machine precision, boundary matching [-i, i]).

## Related tools

- **Diffrax** (JAX-based library providing the numerical differential equation solvers (including symplectic solvers) in which the ReversibleHeun step maps are implemented and exercised) — https://github.com/patrick-kidger/diffrax
- **JAX** (Backend framework supporting autodifferentiation and vectorised convergence/stability sweeps for the step maps) — http://github.com/google/jax

## Examples

```
import diffrax; sol = diffrax.diffeqsolve(diffrax.ODETerm(vector_field), diffrax.ReversibleHeun(), t0=0.0, t1=1.0, dt0=1.0, y0=y0)
```

## Evaluation signals

- Reversibility check: reconstructing y0 from y1 plus the final solver state differs only at floating-point (zero truncation) precision
- Log-log fit of ODE error vs step size yields a slope of ~2 (second order)
- Measured SDE strong convergence order is ~0.5 for general Stratonovich noise and ~1.0 for additive noise
- Dahlquist stability sweep shows the boundary coincides with the complex interval [-i, i] on the imaginary axis
- All validation numbers are recorded in reversible_heun_validation.csv and match the theorems' assertions

## Limitations

- The strong order increases to 1 only under additive noise; general multiplicative noise caps it at 0.5
- The stability region is only the segment [-i, i], so the method is unsuitable for stiff/realdamped (negative real lambda) problems
- Higher-order reversible ODE solvers and hypersolvers during training are not yet available
- Software version numbers are not stated for any tool in the article

## Evidence

- [abstract] The reversible Heun method applied to ODEs is a second-order method whose two-step update coincides with the leapfrog/midpoint method: [evidence span withheld — anchor status: not_found]
- [discussion] Neural SDEs in the article are solved with reversible Heun at unit step size: [evidence span withheld — anchor status: not_found]
