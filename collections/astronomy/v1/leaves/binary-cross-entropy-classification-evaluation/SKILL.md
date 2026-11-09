---
name: binary-cross-entropy-classification-evaluation
description: Use when a two-class, per-timestep sequence classification problem (e.g.
  2D spiral clockwise vs anticlockwise) must be solved by a neural CDE and its held-out
  test accuracy reported against a trained checkpoint.
when_to_use_negative:
- The task is regression or time-series reconstruction (e.g. BIDMC L2-loss experiments)
  rather than two-class classification — cross-entropy is the wrong loss.
- Measurability of the observation path is insufficient for Hermite cubic splines
  with backward differences; use rectilinear interpolation per the article's recommendation
  instead.
- Natural cubic splines are preferred for smoothness reasons — the article reports
  they produce slightly worse results for neural CDEs.
tools:
- name: Diffrax
  role: Solves the neural CDE (reduced to an ODE via ControlTerm.to_ode) with the
    Tsit5 solver and PIDController adaptive step sizing; provides backward_hermite_coefficients
    and CubicInterpolation for the control path
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Autodifferentiation framework in which the Diffrax-based neural CDE training
    loop runs
  repo: http://github.com/google/jax
- name: Equinox
  role: Defines the tanh-squashed MLP vector field / neural CDE model that is trained
    and saved as model.eqx
  repo: https://github.com/patrick-kidger/equinox
- name: Optax
  role: Gradient optimiser used to minimise the binary cross-entropy loss over the
    training split
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:994f41025b7f979dcc30c69da9e6bf233a87510ed42dff4d27e723bdc3e30636
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# binary-cross-entropy-classification-evaluation

## Summary

Trains a neural CDE classifier (e.g. on the 2D spiral clockwise-vs-anticlockwise task) with a binary cross-entropy loss in JAX/Equinox/Optax, then evaluates and visualises test accuracy, saving the trained model checkpoint. This is the reference training-and-evaluation loop behind the neural differential equation experiments in Kidger's thesis.

## When to use

Use when a two-class, per-timestep sequence classification problem (e.g. 2D spiral clockwise vs anticlockwise) must be solved by a neural CDE and its held-out test accuracy reported against a trained checkpoint.

## When NOT to use

- The task is regression or time-series reconstruction (e.g. BIDMC L2-loss experiments) rather than two-class classification — cross-entropy is the wrong loss.
- Measurability of the observation path is insufficient for Hermite cubic splines with backward differences; use rectilinear interpolation per the article's recommendation instead.
- Natural cubic splines are preferred for smoothness reasons — the article reports they produce slightly worse results for neural CDEs.

## Inputs

- 2D spiral binary-classification dataset (train/test splits, clockwise vs anticlockwise labels)
- Observation times ts and values ys per sample for spline interpolation
- Initial hidden state y0 for the neural CDE

## Outputs

- Trained neural CDE checkpoint (model.eqx)
- Test accuracy (target: perfect 100% accuracy on the 2D spiral task)
- Training/test accuracy curves (test_accuracy.png)

## How to apply

Generate the 2D spiral binary-classification train/test splits and normalise each channel to zero mean and unit variance. Build the control path from Hermite cubic splines with backward differences via diffrax.backward_hermite_coefficients, wrap the coefficients in a diffrax.CubicInterpolation, and define the neural CDE with a tanh-squashed MLP vector field. Reduce the CDE to an ODE (ControlTerm.to_ode) and solve with diffrax.diffeqsolve using the Tsit5 solver under a diffrax.PIDController with adaptive rtol/atol, backpropagating via the default RecursiveCheckpointAdjoint (discretise-then-optimise). Optimise the binary cross-entropy loss over the training split with an Optax gradient optimiser, then evaluate accuracy on the held-out test split, save the trained model as model.eqx, and plot training/test accuracy curves.

## Related tools

- **Diffrax** (Solves the neural CDE (reduced to an ODE via ControlTerm.to_ode) with the Tsit5 solver and PIDController adaptive step sizing; provides backward_hermite_coefficients and CubicInterpolation for the control path) — https://github.com/patrick-kidger/diffrax
- **JAX** (Autodifferentiation framework in which the Diffrax-based neural CDE training loop runs) — http://github.com/google/jax
- **Equinox** (Defines the tanh-squashed MLP vector field / neural CDE model that is trained and saved as model.eqx) — https://github.com/patrick-kidger/equinox
- **Optax** (Gradient optimiser used to minimise the binary cross-entropy loss over the training split)

## Examples

```
path = diffrax.CubicInterpolation(diffrax.backward_hermite_coefficients(ts, ys)); sol = diffrax.diffeqsolve(diffrax.ControlTerm(f, path).to_ode(), diffrax.Tsit5(), t0, t1, dt0, y0)
```

## Evaluation signals

- test_accuracy.png exists and shows the test-accuracy curve for the held-out spiral split
- The trained model attains perfect (100%) test accuracy on the 2D spiral binary-classification task
- model.eqx checkpoint file is produced and loadable
- Each channel of the spiral dataset is confirmed normalised to zero mean and unit variance before training
- Training runs with Tsit5 + PIDController under the default RecursiveCheckpointAdjoint (discretise-then-optimise) without solver failures

## Limitations

- Natural cubic splines produce slightly worse results than other interpolation schemes for neural CDEs, for reasons unknown
- Optimise-then-discretise backpropagation may be needed instead of discretise-then-optimise simply to avoid running out of memory on long time series
- The reported 100% accuracy is specific to this simple 2D spiral problem and does not generalise to harder benchmarks

## Evidence

- [discussion] channel-normalisation filter step: "Each channel is normalised to have zero mean and unit variance."
- [discussion] CDE-to-ODE reduction workflow: "The neural CDE was solved by reducing it to an ODE as in Section"
- [discussion] backpropagation mode: "Backpropagation was performed via discretise-then-optimise (Section"
- [discussion] software stack for the experiment: "implemented using the JAX, Equinox, Diffrax, and Optax software libraries"
- [abstract] interpolation choice: "Use Hermite cubic splines with backward differences if possible, due to their smoothness and relatively good measurability. If their measurability properties are insufficient, then use rectilinear"
- [readme] diffrax library role: [evidence span withheld — anchor status: not_found]
