---
name: neural-cde-rnn-correspondence-analysis
description: Use when you need to demonstrate or test that a discrete recurrent architecture
  (e.g. GRU) corresponds to an ODE discretisation, or to interpret gate dynamics in
  continuous time.
when_to_use_negative:
- You need adaptive-step integration accuracy rather than exact recovery of a discrete
  GRU — any solver other than explicit Euler with dt=1 will not reproduce the discrete
  recurrence.
- You are studying architectures whose discrete recurrence is not an explicit Euler
  discretisation (e.g. unrolled second-order or multistep recurrences); the correspondence
  demonstrated here is specific to GRU-style dynamics.
- You need continuous-time dynamics of Neural CDEs solved against a data path; this
  skill only checks the RNN-as-Euler-discretisation equivalence, not CDE interpolation
  (e.g. Hermite cubic splines with backward differences).
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Diffrax
  role: Provides diffrax.ODETerm and diffrax.diffeqsolve with the explicit diffrax.Euler
    solver to integrate the continuous-time GRU ODE at unit step size
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Framework in which the continuous-time GRU vector field and the discrete GRU
    recurrence are implemented and compared step-by-step
  repo: http://github.com/google/jax
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:a22b4ab8d0a694b141f8158bfbf9eef48c3f4242ae05c39102b0d5171acc6168
  source_papers:
  - doi: 10.48550/arxiv.2202.02435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/patrick-kidger/diffrax
  tools:
  - Diffrax
  - JAX
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# neural-cde-rnn-correspondence-analysis

## Summary

Verifies that a standard discrete GRU is exactly the unit-step explicit Euler discretisation of a continuous-time GRU ODE (Example 3.14 of Kidger's 'On Neural Differential Equations'), with the -h(t) term expressing exponential hidden-state decay. This makes concrete the thesis finding that RNNs are discretisations of special cases of neural CDEs.

## When to use

Use when you need to demonstrate or test that a discrete recurrent architecture (e.g. GRU) corresponds to an ODE discretisation, or to interpret gate dynamics in continuous time.

## When NOT to use

- You need adaptive-step integration accuracy rather than exact recovery of a discrete GRU — any solver other than explicit Euler with dt=1 will not reproduce the discrete recurrence.
- You are studying architectures whose discrete recurrence is not an explicit Euler discretisation (e.g. unrolled second-order or multistep recurrences); the correspondence demonstrated here is specific to GRU-style dynamics.
- You need continuous-time dynamics of Neural CDEs solved against a data path; this skill only checks the RNN-as-Euler-discretisation equivalence, not CDE interpolation (e.g. Hermite cubic splines with backward differences).

## Inputs

- Continuous-time GRU vector field definition (gates i(t), r(t), n(t) and hidden-state ODE)
- GRU parameter values (weight matrices and biases for reset/update gates and candidate state)
- Input sequence over an integer time grid
- Initial hidden state h(0)

## Outputs

- Per-step hidden-state trajectories from the unit-step Euler solution of the continuous-time GRU
- Per-step hidden-state trajectories from the discrete GRU recurrence
- Maximum absolute element-wise difference at each timestep (e.g. saved to gru_euler_verification)

## How to apply

Implement the continuous-time GRU vector field with gating variables i(t), r(t), n(t) and hidden-state ODE dh/dt(t) = (1-i(t))*(n(t)-h(t)), explicitly including the -h(t) decay term, wrapped as a diffrax.ODETerm. Solve with diffrax.diffeqsolve using the explicit diffrax.Euler solver with constant unit step size dt=1 over an integer time grid, exploiting Diffrax's default 'discretise then optimise' behaviour. Independently implement the standard discrete GRU recurrence (reset/update gates and candidate hidden state) with identical parameters and inputs. Compare hidden-state trajectories element-wise at every timestep and record the maximum absolute difference; the correspondence holds only if the maximum absolute discrepancy is at machine precision.

## Related tools

- **Diffrax** (Provides diffrax.ODETerm and diffrax.diffeqsolve with the explicit diffrax.Euler solver to integrate the continuous-time GRU ODE at unit step size) — https://github.com/patrick-kidger/diffrax
- **JAX** (Framework in which the continuous-time GRU vector field and the discrete GRU recurrence are implemented and compared step-by-step) — http://github.com/google/jax

## Examples

```
term = diffrax.ODETerm(gru_vector_field); sol = diffrax.diffeqsolve(term, diffrax.Euler(), t0=0, t1=T, dt0=1, y0=h0)
```

## Evaluation signals

- Maximum absolute difference between the unit-step Euler solution and the discrete GRU recurrence is at machine precision (effectively zero) at every timestep.
- Per-step trajectories of both systems are element-wise identical across the whole integer time grid.
- The Euler step of the ODE reproduces exactly h(t+1) = i(t)*h(t) + (1-i(t))*n(t), i.e. the discrete GRU update including the -h(t) exponential-decay term.
- The same parameter values and inputs are used in both implementations, so any discrepancy isolates discretisation error rather than parameter mismatch.
- Both computations are run step-by-step rather than only at the final time, so a single-step offset is detectable early.

## Limitations

- Equivalence holds only for explicit Euler with constant unit step size (dt=1) on an integer time grid; smaller or larger steps, or higher-order solvers, will not match the discrete GRU exactly.
- The -h(t) exponential-decay term explains why GRUs/LSTMs struggle to learn long-term time dependencies — long-horizon behaviour of the continuous-time system decays the hidden state.
- The thesis notes no JAX library existed for computing logsignatures at time of writing, so related CDE-side analyses may require additional tooling.
- Backpropagation via discretise-then-optimise (Diffrax's default) may run out of memory on long time series, where optimise-then-discretise (adjoint) is needed.

## Evidence

- [other] Continuous-time GRU definition and Euler equivalence: [evidence span withheld — anchor status: not_found]
