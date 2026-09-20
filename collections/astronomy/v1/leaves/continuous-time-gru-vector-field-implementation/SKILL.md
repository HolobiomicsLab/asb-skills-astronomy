---
name: continuous-time-gru-vector-field-implementation
description: Use when you need to show a discrete recurrent network (GRU) is the Euler
  discretisation of a continuous-time ODE system, or to inspect the -h(t) exponential-decay
  term governing hidden-state dynamics.
when_to_use_negative:
- When the input is an irregularly sampled time series needing a true continuous-time
  model — use a neural CDE solved with an adaptive solver (e.g. Tsit5) rather than
  a unit-step Euler discretisation of a GRU.
- When the goal is training rather than equivalence verification — machine-precision
  Euler matching is a validation exercise, not a modelling improvement over the discrete
  GRU.
tools:
- name: Diffrax
  role: Provides diffrax.ODETerm and diffrax.diffeqsolve with the diffrax.Euler explicit
    Runge-Kutta solver (dt=1) used to discretise the continuous-time GRU vector field
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Framework in which the continuous-time GRU vector field and the discrete GRU
    recurrence are implemented and compared
  repo: http://github.com/google/jax
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:4c78f77116ffa5746a72d2e238ceaf09d400088d5c84d3dbead019d02b7712c6
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
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# continuous-time-gru-vector-field-implementation

## Summary

Implements the continuous-time GRU of Example 3.14 — gating variables i(t), r(t), n(t) with hidden-state ODE dh/dt = (1-i(t))*(n(t)-h(t)) — as a diffrax.ODETerm in JAX, then verifies that an explicit Euler discretisation with unit step size reproduces the standard discrete GRU recurrence to machine precision.

## When to use

Use when you need to show a discrete recurrent network (GRU) is the Euler discretisation of a continuous-time ODE system, or to inspect the -h(t) exponential-decay term governing hidden-state dynamics.

## When NOT to use

- When the input is an irregularly sampled time series needing a true continuous-time model — use a neural CDE solved with an adaptive solver (e.g. Tsit5) rather than a unit-step Euler discretisation of a GRU.
- When the goal is training rather than equivalence verification — machine-precision Euler matching is a validation exercise, not a modelling improvement over the discrete GRU.

## Inputs

- GRU parameters (update/reset gate and candidate-state weights) shared across the continuous-time and discrete implementations
- Input sequence x(t) over an integer time grid
- Initial hidden state h(0)

## Outputs

- Per-step hidden-state trajectories from the Euler-discretised continuous-time GRU (diffrax.diffeqsolve solution)
- Per-step trajectories from the standard discrete GRU recurrence
- Maximum absolute difference between the two trajectories at each timestep, recorded in gru_euler_verification

## How to apply

Wrap the continuous-time GRU vector field of Example 3.14, including the explicit -h(t) decay term in dh/dt(t) = (1-i(t))*(n(t)-h(t)), as a diffrax.ODETerm in JAX. Solve it with diffrax.diffeqsolve using the diffrax.Euler explicit Runge-Kutta solver with constant unit step size dt=1 over an integer time grid, relying on Diffrax's default discretise-then-optimise behaviour. Independently implement the standard discrete GRU recurrence (reset gate, update gate, candidate hidden state) with identical parameters and inputs. Step both computations forward and compare hidden-state trajectories element-wise at every timestep, recording the maximum absolute discrepancy per step. The check passes only if the unit-step Euler solution matches the discrete GRU at machine precision, confirming the RNNs-are-discretised-CDEs correspondence.

## Related tools

- **Diffrax** (Provides diffrax.ODETerm and diffrax.diffeqsolve with the diffrax.Euler explicit Runge-Kutta solver (dt=1) used to discretise the continuous-time GRU vector field) — https://github.com/patrick-kidger/diffrax
- **JAX** (Framework in which the continuous-time GRU vector field and the discrete GRU recurrence are implemented and compared) — http://github.com/google/jax

## Examples

```
term = diffrax.ODETerm(gru_vector_field); sol = diffrax.diffeqsolve(term, diffrax.Euler(), t0=0, t1=N, dt0=1, y0=h0)
```

## Evaluation signals

- Maximum absolute difference between the unit-step Euler solution and the discrete GRU recurrence is effectively zero (machine precision) at every timestep
- The vector field explicitly contains the -h(t) decay term of dh/dt = (1-i(t))*(n(t)-h(t)); omitting it breaks the equivalence
- Per-step trajectories from both implementations are stored element-wise, allowing element-wise diff at every timestep
- Identical parameter values and inputs are used for both the continuous-time ODE and the discrete recurrence

## Limitations

- The equivalence holds only for the explicit Euler solver with constant unit step size (dt=1) on an integer time grid; other solvers or step sizes will not reproduce the discrete GRU exactly.
- The -h(t) term corresponds to exponential decay of the hidden state, which explains the classic fact that GRUs/LSTMs struggle to learn long-term time dependencies — the continuous-time form does not remove this behaviour.

## Evidence

- [abstract] Example 3.14 continuous-time GRU: [evidence span withheld — anchor status: not_found]
- [abstract] RNNs as discretisations: "Many popular neural network architectures, such as residual networks and recurrent networks, are discretisations."
- [abstract] decay term interpretation: "This explains the classic fact that GRUs/LSTMs struggle to learn long-term time dependencies."
- [readme] Diffrax role: "Diffrax is a [JAX](https://github.com/google/jax)-based library providing numerical differential equation solvers."
