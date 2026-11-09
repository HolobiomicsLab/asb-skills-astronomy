---
name: adaptive-ode-solving-with-tsit5
description: Use when a JAX/Equinox neural ODE or neural CDE hidden state must be
  integrated over an interpolated control path (e.g. Hermite cubic splines over 2D
  spiral classification data) with adaptive step-size control rather than a fixed
  grid.
when_to_use_negative:
- 'When solving neural SDEs: the thesis instead solves those with the reversible Heun
  method with unit step size, not Tsit5 adaptive ODE control'
- 'When the data must be preprocessed into logsignatures over windows: at the time
  of writing there does not exist a JAX library for computing logsignatures'
- When you are committed to the PyTorch ecosystem (torchdiffeq/torchcde) rather than
  the JAX/Diffrax stack this workflow is built on
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Diffrax
  role: Provides diffeqsolve, the Tsit5 solver, PIDController adaptive step-size control,
    backward_hermite_coefficients, CubicInterpolation, and ControlTerm.to_ode for
    the ODE-reduced CDE
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Autodifferentiation framework that Diffrax is written for; supports GPU-capable,
    vmappable solving
  repo: http://github.com/google/jax
- name: Equinox
  role: Defines the tanh-squashed MLP vector field of the neural CDE as a JAX-compatible
    model
  repo: https://github.com/patrick-kidger/equinox
- name: Optax
  role: Gradient optimiser used to train the neural CDE with binary cross-entropy
    loss
  repo: https://github.com/google-deepmind/optax
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:9a18770c8d49cc78da2bdba127effe7a2eb90931fc8690dafac653354b451ba1
  source_papers:
  - doi: 10.48550/arxiv.2202.02435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/patrick-kidger/diffrax
  tools:
  - Diffrax
  - Equinox
  - JAX
  - Optax
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# Adaptive ODE solving with Tsit5

## Summary

Integrates neural ODEs/CDEs in JAX by reducing a neural CDE to an ODE and solving it with diffrax.diffeqsolve under the Tsit5 solver with adaptive PIDController step-size control, as used for the 2D spiral binary-classification experiment in Kidger's thesis.

## When to use

Use when a JAX/Equinox neural ODE or neural CDE hidden state must be integrated over an interpolated control path (e.g. Hermite cubic splines over 2D spiral classification data) with adaptive step-size control rather than a fixed grid.

## When NOT to use

- When solving neural SDEs: the thesis instead solves those with the reversible Heun method with unit step size, not Tsit5 adaptive ODE control
- When the data must be preprocessed into logsignatures over windows: at the time of writing there does not exist a JAX library for computing logsignatures
- When you are committed to the PyTorch ecosystem (torchdiffeq/torchcde) rather than the JAX/Diffrax stack this workflow is built on

## Inputs

- Per-channel time series with observations at timestamps ts and values ys (e.g. the 2D spiral binary-classification train/test splits, clockwise vs anticlockwise)
- An Equinox neural CDE vector field (tanh-squased MLP)
- Chosen solver tolerances (rtol/atol) for the PIDController

## Outputs

- A diffrax Solution containing the integrated hidden state of the neural CDE
- Trained model checkpoint (model.eqx) and training/test accuracy curves (test_accuracy.png) for the spiral task

## How to apply

Normalise each channel of the input series to zero mean and unit variance, then build the control path with Hermite cubic splines with backward differences via diffrax.backward_hermite_coefficients wrapped in a diffrax.CubicInterpolation. Define the neural CDE vector field as a tanh-squashed MLP in Equinox and reduce the CDE to an ODE (e.g. ControlTerm.to_ode / ODETerm). Call diffrax.diffeqsolve with the Tsit5 solver and a diffrax.PIDController stepsize_controller supplying the chosen rtol/atol, letting the controller adapt the step size; keep the default RecursiveCheckpointAdjoint (discretise-then-optimise) for backpropagation. Train with binary cross-entropy using an Optax optimiser, evaluate on the held-out test split, and save the trained model checkpoint (model.eqx). The rationale: Hermite cubic splines with backward differences are recommended for smoothness and measurability, and Tsit5 with PID control gives an accurate adaptive solve of the ODE-reduced CDE.

## Related tools

- **Diffrax** (Provides diffeqsolve, the Tsit5 solver, PIDController adaptive step-size control, backward_hermite_coefficients, CubicInterpolation, and ControlTerm.to_ode for the ODE-reduced CDE) — https://github.com/patrick-kidger/diffrax
- **JAX** (Autodifferentiation framework that Diffrax is written for; supports GPU-capable, vmappable solving) — http://github.com/google/jax
- **Equinox** (Defines the tanh-squashed MLP vector field of the neural CDE as a JAX-compatible model) — https://github.com/patrick-kidger/equinox
- **Optax** (Gradient optimiser used to train the neural CDE with binary cross-entropy loss) — https://github.com/google-deepmind/optax

## Examples

```
sol = diffrax.diffeqsolve(ODETerm(vector_field), diffrax.Tsit5(), t0, t1, None, y0, stepsize_controller=diffrax.PIDController(rtol=1e-3, atol=1e-6))
```

## Evaluation signals

- The trained neural CDE attains perfect (100%) test accuracy on the 2D spiral binary-classification task, matching the reported finding
- The validation figure test_accuracy.png exists after the run
- diffeqsolve completes with solver=Tsit5 and stepsize_controller=PIDController without adaptive step-size failures; tightening rtol/atol yields a consistent solution
- Each input channel is normalised to zero mean and unit variance before spline coefficients are computed
- Backpropagation runs via the default RecursiveCheckpointAdjoint (discretise-then-optimise) and the checkpoint model.eqx is saved

## Limitations

- Discretise-then-optimise backpropagation is the default; optimise-then-discretise was needed in the thesis simply to avoid running out of memory on long time series
- Neural CDEs that use natural cubic splines produce slightly worse results than other interpolation schemes; prefer Hermite cubic splines with backward differences (or rectilinear if measurability is insufficient)
- At the time of writing there does not exist a JAX library for computing logsignatures, limiting logsignature preprocessing within this JAX workflow
- Software version numbers are not stated for any tool, so reproducibility of exact solver behaviour is not guaranteed

## Evidence

- [other] Adaptive Tsit5 solving under PID control is prescribed by the task workflow: [evidence span withheld — anchor status: not_found]
- [discussion] The neural CDE is reduced to an ODE before solving: [evidence span withheld — anchor status: not_found]
- [discussion] Backpropagation mode for the adaptive solve: "Backpropagation was performed via discretise-then-optimise (Section"
- [abstract] Hermite cubic splines with backward differences are the recommended interpolation for the control path: "Use Hermite cubic splines with backward differences if possible, due to their smoothness and relatively good measurability."
- [readme] Tsit5 is among the solvers Diffrax provides: "lots of different solvers (including `Tsit5`, `Dopri8`, symplectic solvers, implicit solvers);"
