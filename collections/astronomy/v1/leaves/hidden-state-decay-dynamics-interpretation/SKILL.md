---
name: hidden-state-decay-dynamics-interpretation
description: Use when you need to demonstrate that a discrete recurrent architecture
  (GRU) is the Euler discretisation of a continuous-time system, or to interpret hidden-state
  dynamics such as the exponential decay implied by the -h(t) term.
when_to_use_negative:
- When the goal is to train or fine-tune a GRU rather than to verify its continuous-time
  ODE correspondence — this is a verification/interpretation procedure, not a learning
  pipeline
- When the recurrence in question is not recoverable by explicit Euler discretisation
  at unit step size (e.g. architectures with adaptive or multi-step updates) — the
  machine-precision equivalence check will fail by construction
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Diffrax
  role: Provides diffrax.ODETerm, diffrax.Euler explicit solver and diffrax.diffeqsolve
    to integrate the continuous-time GRU vector field at unit step size
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Framework in which the continuous-time GRU vector field and the discrete GRU
    recurrence are implemented and compared step-by-step
  repo: http://github.com/google/jax
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:3df75be638f95f583d200036506b6a170d2f58dab80ab1317bb5e8e0bf9fe031
  source_papers:
  - doi: 10.48550/arxiv.2202.02435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/patrick-kidger/diffrax
  tools:
  - Diffrax
  - JAX
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# hidden-state-decay-dynamics-interpretation

## Summary

Reconstructs the continuous-time GRU ODE of Example 3.14 (gating variables i(t), r(t), n(t) with dh/dt(t) = (1-i(t))*(n(t)-h(t))) and verifies that the standard discrete GRU recurrence is its explicit Euler discretisation at unit step size, using JAX and Diffrax.

## When to use

Use when you need to demonstrate that a discrete recurrent architecture (GRU) is the Euler discretisation of a continuous-time system, or to interpret hidden-state dynamics such as the exponential decay implied by the -h(t) term.

## When NOT to use

- When the goal is to train or fine-tune a GRU rather than to verify its continuous-time ODE correspondence — this is a verification/interpretation procedure, not a learning pipeline
- When the recurrence in question is not recoverable by explicit Euler discretisation at unit step size (e.g. architectures with adaptive or multi-step updates) — the machine-precision equivalence check will fail by construction

## Inputs

- GRU parameter values (weight matrices and biases for reset/update gates and candidate state)
- Input sequence x(t) at each timestep
- Initial hidden state h(0)

## Outputs

- Per-step hidden-state trajectories from the Euler-discretised continuous-time GRU (written to gru_euler_verification)
- Per-step hidden-state trajectories from the discrete GRU recurrence
- Maximum absolute difference between the two trajectories at each timestep (must be at machine precision)

## How to apply

Implement the continuous-time GRU vector field of Example 3.14 in JAX, wrapping it as a diffrax.ODETerm and explicitly including the -h(t) decay term of the hidden state, which per Remark 3.15 corresponds to exponential decay of the GRU hidden state. Solve the system with diffrax.diffeqsolve using the diffrax.Euler explicit solver with constant unit step size (dt=1) over an integer time grid, keeping Diffrax's default discretise-then-optimise behaviour. In parallel, implement the standard discrete GRU recurrence (reset/update gates and candidate hidden state) with identical parameters and inputs, then compare hidden-state trajectories element-wise at every timestep. Validation passes only if the maximum absolute discrepancy between the Euler solution and the discrete GRU is at machine precision (effectively zero), confirming the RNNs-are-discretised-CDEs correspondence.

## Related tools

- **Diffrax** (Provides diffrax.ODETerm, diffrax.Euler explicit solver and diffrax.diffeqsolve to integrate the continuous-time GRU vector field at unit step size) — https://github.com/patrick-kidger/diffrax
- **JAX** (Framework in which the continuous-time GRU vector field and the discrete GRU recurrence are implemented and compared step-by-step) — http://github.com/google/jax

## Examples

```
term = diffrax.ODETerm(vector_field); sol = diffrax.diffeqsolve(term, diffrax.Euler(), t0=0, t1=T, dt0=1, y0=h0, args=params)
```

## Evaluation signals

- Maximum absolute difference between the unit-step Euler solution and the discrete GRU recurrence is at machine precision (effectively zero) at every timestep
- The hidden-state vector field explicitly contains the -h(t) term; omitting it breaks the equivalence with the discrete GRU update
- Per-step trajectories of both computations agree element-wise when compared step-by-step with identical parameters and inputs
- The solver is diffrax.Euler with constant dt=1 over an integer time grid — smaller or adaptive steps are not the GRU correspondence

## Limitations

- The equivalence holds only for explicit Euler discretisation with unit step size; other solvers or step sizes will not reproduce the discrete GRU exactly
- The -h(t) exponential decay term explains why GRUs/LSTMs struggle to learn long-term time dependencies, so the continuous-time view exposes rather than removes this limitation
- Diffrax's default discretise-then-optimise behaviour must be retained for the correspondence; changing the adjoint strategy is outside this verification

## Evidence

- [abstract] Continuous-time GRU ODE definition: [evidence span withheld — anchor status: not_found]
- [abstract] Decay interpretation (Remark 3.15): [evidence span withheld — anchor status: not_found]
- [discussion] Verification workflow and success criterion: [evidence span withheld — anchor status: not_found]
