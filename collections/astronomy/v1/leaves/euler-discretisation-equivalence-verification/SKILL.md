---
name: euler-discretisation-equivalence-verification
description: Use when you need to numerically confirm that a discrete recurrent architecture
  (GRU-style gating with update/reset gates and candidate state) matches a proposed
  continuous-time ODE under unit-step explicit Euler discretisation.
when_to_use_negative:
- When you want approximate continuous-time behaviour (e.g. irregularly sampled time
  series) rather than exact unit-step equivalence — use an adaptive solver like Tsit5
  instead of Euler at dt=1.
- When verifying architectures whose continuous-time analogue is not known in closed
  form (e.g. LSTMs without a derived ODE), since there is no target vector field to
  compare against.
- When inputs are continuous paths requiring CDE/interpolation treatment rather than
  discrete sequences fed to a GRU recurrence.
tools:
- name: Diffrax
  role: Provides diffrax.ODETerm and diffrax.Euler (explicit Runge-Kutta) used with
    diffrax.diffeqsolve at constant dt=1 to Euler-discretise the continuous-time GRU
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Framework in which the continuous-time GRU vector field and the discrete GRU
    recurrence are implemented and compared element-wise
  repo: http://github.com/google/jax
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:7c7484f61ef53a4a9e79ee51533c4bdee877797862e30bfa660731c9e73b2406
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
yes_rate: 0.5
n_claims: 4
---

# Euler-discretisation equivalence verification

## Summary

Verifies that a standard discrete GRU recurrence is exactly the explicit Euler discretisation of the continuous-time GRU ODE of Example 3.14, whose hidden-state equation dh/dt(t) = (1-i(t))*(n(t)-h(t)) includes an exponential-decay term. This confirms the 'RNNs are discretised CDEs' correspondence numerically, at machine precision.

## When to use

Use when you need to numerically confirm that a discrete recurrent architecture (GRU-style gating with update/reset gates and candidate state) matches a proposed continuous-time ODE under unit-step explicit Euler discretisation.

## When NOT to use

- When you want approximate continuous-time behaviour (e.g. irregularly sampled time series) rather than exact unit-step equivalence — use an adaptive solver like Tsit5 instead of Euler at dt=1.
- When verifying architectures whose continuous-time analogue is not known in closed form (e.g. LSTMs without a derived ODE), since there is no target vector field to compare against.
- When inputs are continuous paths requiring CDE/interpolation treatment rather than discrete sequences fed to a GRU recurrence.

## Inputs

- Continuous-time GRU vector field parameters (weights for gates i(t), r(t), n(t)) as a JAX pytree
- Initial hidden state h(0) as a JAX array
- Sequence of discrete inputs (e.g. x(t) per timestep) shared by both the ODE solver and the discrete GRU
- Integer time grid (t0=0, unit steps, dt=1)

## Outputs

- gru_euler_verification table/file with per-step hidden-state trajectories for both the Euler-discretised continuous-time GRU and the discrete GRU
- Per-step maximum absolute difference between the two trajectories
- Pass/fail validation flag (pass iff maximum discrepancy is at machine precision)

## How to apply

Implement the continuous-time GRU vector field of Example 3.14 in JAX — with gating variables i(t), r(t), n(t) and the hidden-state equation dh/dt(t) = (1-i(t))*(n(t)-h(t)), explicitly including the -h(t) decay term — and wrap it as a diffrax.ODETerm. Solve with diffrax.diffeqsolve using the diffrax.Euler explicit Runge-Kutta solver at constant unit step size (dt=1) over an integer time grid, relying on Diffrax's default 'discretise-then-optimise' behaviour. In parallel, implement the standard discrete GRU recurrence (reset gate, update gate, candidate hidden state) with the same parameters and inputs. Step both computations forward and compare hidden-state trajectories element-wise at every timestep, recording the maximum absolute difference per step. The check passes only if the maximum absolute discrepancy is at machine precision (effectively zero).

## Related tools

- **Diffrax** (Provides diffrax.ODETerm and diffrax.Euler (explicit Runge-Kutta) used with diffrax.diffeqsolve at constant dt=1 to Euler-discretise the continuous-time GRU) — https://github.com/patrick-kidger/diffrax
- **JAX** (Framework in which the continuous-time GRU vector field and the discrete GRU recurrence are implemented and compared element-wise) — http://github.com/google/jax

## Examples

```
sol = diffrax.diffeqsolve(diffrax.ODETerm(gru_vector_field), diffrax.Euler(), t0=0, t1=N, dt0=1, y0=h0); max|sol.ys - gru_discrete| == 0
```

## Evaluation signals

- Maximum absolute element-wise difference between the Euler solution and the discrete GRU is effectively zero (machine precision) at every timestep.
- The continuous-time vector field explicitly contains the -h(t) term, matching dh/dt(t) = (1-i(t))*(n(t)-h(t)) from Example 3.14.
- Solver configuration matches the required discretisation: diffrax.Euler solver, constant unit step size dt=1, integer time grid.
- Per-step trajectories are recorded for both systems and are identical element-wise, not merely close (e.g. not just within a loose tolerance).
- The same parameter values and input sequence are used for both the ODE solve and the discrete recurrence.

## Limitations

- Equivalence holds only for explicit Euler at unit step size; using a different solver or smaller step will not reproduce the discrete GRU exactly.
- The -h(t) decay term explains why GRUs/LSTMs struggle to learn long-term time dependencies, so the ODE form inherits this limitation for long sequences.
- The equivalence relies on Diffrax's default 'discretise-then-optimise' behaviour; changing backpropagation mode does not affect forward equivalence but changes gradients.

## Evidence

- [abstract] Continuous-time GRU ODE definition: [evidence span withheld — anchor status: not_found]
- [abstract] Architectures as discretisations: "Many popular neural network architectures, such as residual networks and recurrent networks, are discretisations."
- [methods] Solver configuration in workflow: [evidence span withheld — anchor status: not_found]
