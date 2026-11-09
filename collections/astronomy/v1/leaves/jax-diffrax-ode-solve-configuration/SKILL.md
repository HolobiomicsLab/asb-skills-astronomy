---
name: jax-diffrax-ode-solve-configuration
description: Use when a numerical differential-equation model must be solved in JAX
  — e.g. verifying that a discrete recurrence (GRU) matches an Euler-discretised continuous-time
  system, or training a neural CDE on interpolated control paths with adaptive step
  sizes.
when_to_use_negative:
- When the target is training a continuous model where solution accuracy matters —
  unit-step diffrax.Euler will not match Tsit5's adaptive accuracy
- When you need exact reproduction of a discrete recurrence and the solver is not
  explicit Euler with matching unit step size — other solvers (e.g. Tsit5) will not
  coincide with the discrete recurrence
- When the problem is stochastic or controlled and requires SDE/CDE solvers (torchsde,
  torchcde) rather than a plain ODETerm
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Diffrax
  role: Provides diffrax.diffeqsolve, diffrax.ODETerm, diffrax.Euler, diffrax.Tsit5,
    diffrax.PIDController, diffrax.backward_hermite_coefficients and CubicInterpolation
    used to configure and execute the ODE/CDE solves
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Framework in which the Diffrax library is written; supplies autodifferentiation
    and vectorisation for the vector fields and training loops
  repo: http://github.com/google/jax
- name: torchdiffeq
  role: PyTorch alternative providing differentiable ODE solvers with adjoint backpropagation,
    mentioned as the analogue of Diffrax in the PyTorch ecosystem
  repo: https://github.com/rtqichen/torchdiffeq
- name: torchcde
  role: PyTorch prototype library for CDE solvers; author recommends Diffrax for new
    projects instead
  repo: https://github.com/patrick-kidger/torchcde
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_content_hash: sha256:de2a61a367e8cff06f8fc90a1fb825eb4d4f83bea832a16955567f8f877f9c23
  source_papers:
  - doi: 10.48550/arxiv.2202.02435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/patrick-kidger/diffrax
  tools:
  - Diffrax
  - JAX
  - torchcde
  - torchdiffeq
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# jax-diffrax-ode-solve-configuration

## Summary

Configure and run ODE (and reduced-CDE) solves in Diffrax on JAX, choosing the right solver, step-size control, and backpropagation strategy (e.g. diffrax.Euler with constant unit dt=1 for exact GRU-equivalence checks vs diffrax.Tsit5 with diffrax.PIDController for adaptive neural CDE training).

## When to use

Use when a numerical differential-equation model must be solved in JAX — e.g. verifying that a discrete recurrence (GRU) matches an Euler-discretised continuous-time system, or training a neural CDE on interpolated control paths with adaptive step sizes.

## When NOT to use

- When the target is training a continuous model where solution accuracy matters — unit-step diffrax.Euler will not match Tsit5's adaptive accuracy
- When you need exact reproduction of a discrete recurrence and the solver is not explicit Euler with matching unit step size — other solvers (e.g. Tsit5) will not coincide with the discrete recurrence
- When the problem is stochastic or controlled and requires SDE/CDE solvers (torchsde, torchcde) rather than a plain ODETerm

## Inputs

- Vector field (e.g. continuous-time GRU with gating variables i(t), r(t), n(t)) wrapped as a diffrax.ODETerm
- Control path coefficients from diffrax.backward_hermite_coefficients wrapped in a diffrax.CubicInterpolation (for CDE-to-ODE reduction)
- Solver specification (diffrax.Euler for unit-step discretisation; diffrax.Tsit5 for adaptive solving)
- Step-size / tolerance configuration (constant dt=1, or diffrax.PIDController with chosen rtol/atol)

## Outputs

- Hidden-state trajectories at every timestep from diffrax.diffeqsolve
- Maximum absolute difference between Euler-discretised continuous-time GRU and discrete GRU per step (gru_euler_verification)
- Trained neural CDE solution used for classification (e.g. model.eqx checkpoint downstream)

## How to apply

First decide the discretisation regime: for an exact correspondence check (continuous-time GRU of Example 3.14, dh/dt = (1-i(t))*(n(t)-h(t)) with explicit -h(t) decay), wrap the vector field as a diffrax.ODETerm and solve with diffrax.diffeqsolve using the diffrax.Euler solver at constant unit step size (dt=1) over an integer time grid, relying on Diffrax's default 'discretise then optimise' behaviour so the trajectory matches the discrete GRU recurrence at machine precision. For learning tasks (e.g. 2D spiral classification), build the control path with diffrax.backward_hermite_coefficients into a diffrax.CubicInterpolation, reduce the CDE to an ODE, and solve with the Tsit5 solver under adaptive step-size control via diffrax.PIDController (chosen rtol/atol), again using the default RecursiveCheckpointAdjoint discretise-then-optimise backpropagation. Choose constant-step Euler only when step-by-step equivalence to a discrete recurrence is the target; choose adaptive Tsit5 when accuracy of the continuous solution matters during training. Record per-step trajectories and maximum absolute differences to validate the configuration.

## Related tools

- **Diffrax** (Provides diffrax.diffeqsolve, diffrax.ODETerm, diffrax.Euler, diffrax.Tsit5, diffrax.PIDController, diffrax.backward_hermite_coefficients and CubicInterpolation used to configure and execute the ODE/CDE solves) — https://github.com/patrick-kidger/diffrax
- **JAX** (Framework in which the Diffrax library is written; supplies autodifferentiation and vectorisation for the vector fields and training loops) — http://github.com/google/jax
- **torchdiffeq** (PyTorch alternative providing differentiable ODE solvers with adjoint backpropagation, mentioned as the analogue of Diffrax in the PyTorch ecosystem) — https://github.com/rtqichen/torchdiffeq
- **torchcde** (PyTorch prototype library for CDE solvers; author recommends Diffrax for new projects instead) — https://github.com/patrick-kidger/torchcde

## Examples

```
sol = diffrax.diffeqsolve(diffrax.ODETerm(vector_field), solver=diffrax.Euler(), t0=0, t1=T, dt0=1.0, y0=h0, stepsize_controller=diffrax.ConstantStepSize())
```

## Evaluation signals

- The maximum absolute discrepancy between the unit-step Euler solution of the continuous-time GRU and the discrete GRU recurrence is at machine precision (effectively zero) at every timestep
- For the spiral classification task, the trained model attains 100% test accuracy and a test_accuracy.png figure exists
- The solver configuration matches intent: diffrax.Euler with constant dt=1 for discretisation-equivalence checks; Tsit5 with diffrax.PIDController (rtol/atol) for adaptive training solves
- Per-step trajectories are recorded for both the Euler-discretised and discrete computations, allowing element-wise comparison at every timestep
- Backpropagation uses Diffrax's default discretise-then-optimise (RecursiveCheckpointAdjoint) behaviour as specified

## Limitations

- Software version numbers are not stated for any tool in the article
- The exact Euler-equivalence holds only for the specific continuous-time GRU system (with the -h(t) exponential-decay term); other recurrences require their own derivation
- Hermite cubic splines with backward differences are recommended over natural cubic splines, which 'produce slightly worse results than those using other interpolation schemes'

## Evidence

- [abstract] Example 3.14 defines the continuous-time GRU with gating variables i(t), r(t), n(t) and hidden-state equation: [evidence span withheld — anchor status: not_found]
- [abstract] the standard GRU recurrence is an explicit Euler discretisation: [evidence span withheld — anchor status: not_found]
- [intro] Diffrax written for JAX: [evidence span withheld — anchor status: not_found]
