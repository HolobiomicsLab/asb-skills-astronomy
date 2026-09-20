---
name: neural-cde-training
description: Use when training a model on irregularly or continuously indexed time-series
  data (e.g. 2D spiral classification, BIDMC vital-sign regression) where a CDE hidden
  state driven by an interpolated path is the target architecture.
when_to_use_negative:
- When the data is not a time series / control path (e.g. static feature tables) —
  a CDE has no continuous driving path to interpolate
- When natural cubic splines are the only interpolation available and top accuracy
  is required — the thesis reports neural CDEs using natural cubic splines produce
  slightly worse results than other interpolation schemes
- When you need logsignature preprocessing in JAX — at the time of writing there does
  not exist a JAX library for computing logsignatures; use the PyTorch/torchcde route
  instead
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Diffrax
  role: JAX library providing the CDE/ODE solvers (Tsit5, PIDController, backward_hermite_coefficients,
    CubicInterpolation, RecursiveCheckpointAdjoint) used to solve and differentiate
    the neural CDE
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Autodifferentiation framework underpinning the Diffrax-based neural CDE implementation
  repo: http://github.com/google/jax
- name: Equinox
  role: JAX neural-network library used to define the tanh-squashed MLP vector field
    of the CDE
  repo: https://github.com/patrick-kidger/equinox
- name: Optax
  role: Gradient optimiser library used with JAX/Equinox to train the CDE with binary
    cross-entropy or L2 loss
- name: torchcde
  role: PyTorch prototype library providing differentiable CDE solvers and path interpolation
    for the neural RDE/CDE experiments on BIDMC
  repo: https://github.com/patrick-kidger/torchcde
- name: torchdiffeq
  role: PyTorch ODE solver library used for the ODE-RNN baseline and adjoint backpropagation
  repo: https://github.com/rtqichen/torchdiffeq
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_content_hash: sha256:4c7ac61df457e391a1e30499727d4b473d368715d83eb8ca49c9463a75fff277
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
  - PyTorch
  - torchcde
  - torchdiffeq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# neural-cde-training

## Summary

Trains neural controlled differential equations (neural CDEs) on time-series tasks by interpolating irregularly sampled data into a continuous control path (Hermite cubic splines with backward differences, or log-signature preprocessing) and solving the CDE by reduction to an ODE with an adaptive-step solver.

## When to use

Use when training a model on irregularly or continuously indexed time-series data (e.g. 2D spiral classification, BIDMC vital-sign regression) where a CDE hidden state driven by an interpolated path is the target architecture.

## When NOT to use

- When the data is not a time series / control path (e.g. static feature tables) — a CDE has no continuous driving path to interpolate
- When natural cubic splines are the only interpolation available and top accuracy is required — the thesis reports neural CDEs using natural cubic splines produce slightly worse results than other interpolation schemes
- When you need logsignature preprocessing in JAX — at the time of writing there does not exist a JAX library for computing logsignatures; use the PyTorch/torchcde route instead

## Inputs

- Irregularly or densely sampled multivariate time series with train/test splits (e.g. 2D spiral binary-classification data, BIDMC respiratory rate / heart rate / SpO2 series from the TSR archive)
- Channel-wise normalisation statistics (zero mean, unit variance)
- Model hyperparameters: MLP vector-field width/depth, solver rtol/atol for PIDController, interpolation scheme (backward-difference Hermite cubic) or log-ODE step sizes (8/128/512) with logsignature depth 2-3

## Outputs

- Trained neural CDE model checkpoint (e.g. model.eqx)
- Test-set metric: binary classification accuracy (100% on the 2D spiral task) or test L2 loss (mean ± std over seeds)
- Training/test accuracy (or loss) curves, e.g. test_accuracy.png
- Wall-clock training time for comparison against ODE-RNN / neural RDE baselines

## How to apply

Normalise each channel of the dataset to zero mean and unit variance, then build the control path: compute coefficients with diffrax.backward_hermite_coefficients and wrap them in a diffrax.CubicInterpolation, preferring Hermite cubic splines with backward differences over natural cubic splines (which empirically produce slightly worse results). Define the neural CDE vector field as a tanh-squashed MLP (e.g. with Equinox) and reduce the CDE to an ODE via ControlTerm.to_ode so it can be solved with diffrax.diffeqsolve using the Tsit5 solver under adaptive step-size control (diffrax.PIDController with chosen rtol/atol). Backpropagate via the default RecursiveCheckpointAdjoint, i.e. discretise-then-optimise. Train with binary cross-entropy (spiral classification) or L2 loss (BIDMC regression) using an Optax optimiser, then evaluate on the held-out test split. For very long series where integration steps exceed the sampling rate, preprocess windows into depth 2-3 logsignatures and interpolate them into the driving path instead.

## Related tools

- **Diffrax** (JAX library providing the CDE/ODE solvers (Tsit5, PIDController, backward_hermite_coefficients, CubicInterpolation, RecursiveCheckpointAdjoint) used to solve and differentiate the neural CDE) — https://github.com/patrick-kidger/diffrax
- **JAX** (Autodifferentiation framework underpinning the Diffrax-based neural CDE implementation) — http://github.com/google/jax
- **Equinox** (JAX neural-network library used to define the tanh-squashed MLP vector field of the CDE) — https://github.com/patrick-kidger/equinox
- **Optax** (Gradient optimiser library used with JAX/Equinox to train the CDE with binary cross-entropy or L2 loss)
- **torchcde** (PyTorch prototype library providing differentiable CDE solvers and path interpolation for the neural RDE/CDE experiments on BIDMC) — https://github.com/patrick-kidger/torchcde
- **torchdiffeq** (PyTorch ODE solver library used for the ODE-RNN baseline and adjoint backpropagation) — https://github.com/rtqichen/torchdiffeq

## Examples

```
coeffs = diffrax.backward_hermite_coefficients(ts, xs); path = diffrax.CubicInterpolation(coeffs); sol = diffrax.diffeqsolve(diffrax.ControlTerm(vector_field, path).to_ode(), diffrax.Tsit5(), t0, t1, None, y0, stepsize_controller=diffrax.PIDController(rtol=1e-3, atol=1e-6))
```

## Evaluation signals

- Trained model attains perfect (100%) test accuracy on the 2D spiral binary-classification task, and test_accuracy.png exists
- On BIDMC tasks, test L2 loss falls within the reported mean ± standard deviation of the neural CDE / ODE-RNN baselines
- All channels of the input data are verified to have zero mean and unit variance before path construction
- The control path is a diffrax.CubicInterpolation built from backward_hermite_coefficients (or a logsignature-interpolated path), not a natural cubic spline
- Backpropagation uses the discretise-then-optimise route (RecursiveCheckpointAdjoint); neural RDE should show ~10x wall-clock training speed-up over the neural CDE baseline

## Limitations

- Natural cubic splines produce slightly worse neural CDE results than other interpolation schemes for unknown reasons
- At the time of writing there does not exist a JAX library for computing logsignatures, limiting log-ODE preprocessing to PyTorch/torchcde
- Optimise-then-discretise backpropagation may be needed simply to avoid running out of memory on long time series, trading memory against the default discretise-then-optimise adjoint
- GRU/LSTM-style architectures correspond to explicit Euler discretisations and the -h(t) term gives exponential decay of the hidden state, explaining their difficulty with long-term time dependencies relative to CDEs

## Evidence

- [readme] torchcde role: [evidence span withheld — anchor status: not_found]
