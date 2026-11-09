---
name: hermite-cubic-spline-interpolation
description: Use when irregularly sampled time series must be turned into a continuous
  control path for a neural CDE, especially the 2D spiral classification setup, and
  a smooth interpolation with good measurability is needed.
when_to_use_negative:
- The task involves a smooth, densely sampled path where rectilinear (piecewise-linear)
  interpolation suffices — the thesis recommends rectilinear 'if their measurability
  properties are insufficient'.
- The pipeline is PyTorch-based and cannot use JAX; torchcde (the prototype library)
  would be the alternative, though its author recommends Diffrax for new projects.
- A natural cubic spline is prescribed by an existing protocol — the thesis reports
  neural CDEs with natural cubic splines produce slightly worse results than other
  interpolation schemes.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: Diffrax
  role: Provides backward_hermite_coefficients and CubicInterpolation to build the
    Hermite cubic control path for the neural CDE
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Autodifferentiation framework in which Diffrax is written; enables discretise-then-optimise
    backpropagation through the interpolated path
  repo: http://github.com/google/jax
- name: torchcde
  role: Prototype PyTorch alternative for CDE interpolation/solving; author recommends
    Diffrax for new projects
  repo: https://github.com/patrick-kidger/torchcde
- name: Equinox
  role: Defines the tanh-squashed MLP vector field of the neural CDE as an Equinox
    module on the interpolated path
  repo: https://github.com/patrick-kidger/equinox
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:e7a4f1d48cfb1b496b8ee0de81d489fc90a56c87019152a935ae3bd508a1de24
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
  - torchcde
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hermite-cubic-spline-interpolation

## Summary

Builds the smooth control path X(t) that drives a neural CDE by computing Hermite cubic spline coefficients with backward differences from irregularly sampled observations, using diffrax.backward_hermite_coefficients and diffrax.CubicInterpolation in JAX. This interpolation scheme is recommended over natural cubic splines, which empirically degrade neural CDE results.

## When to use

Use when irregularly sampled time series must be turned into a continuous control path for a neural CDE, especially the 2D spiral classification setup, and a smooth interpolation with good measurability is needed.

## When NOT to use

- The task involves a smooth, densely sampled path where rectilinear (piecewise-linear) interpolation suffices — the thesis recommends rectilinear 'if their measurability properties are insufficient'.
- The pipeline is PyTorch-based and cannot use JAX; torchcde (the prototype library) would be the alternative, though its author recommends Diffrax for new projects.
- A natural cubic spline is prescribed by an existing protocol — the thesis reports neural CDEs with natural cubic splines produce slightly worse results than other interpolation schemes.

## Inputs

- Irregularly sampled time series (observation times, values, e.g. the 2D spiral binary-classification train/test splits)
- Per-channel normalisation statistics (zero mean, unit variance)
- Optional data augmentations/timestamps for the spline construction

## Outputs

- Spline coefficient array (from diffrax.backward_hermite_coefficients)
- diffrax.CubicInterpolation control path object consumable by a neural CDE ControlTerm
- Trained model checkpoint (model.eqx) and test-accuracy figures (e.g. test_accuracy.png)

## How to apply

First normalise each channel of the dataset (e.g. the 2D spiral binary-classification data, clockwise vs anticlockwise) to zero mean and unit variance. Pass the observation times, values, and augmentations to diffrax.backward_hermite_coefficients to obtain the spline coefficients, then wrap them in a diffrax.CubicInterpolation to form the control path. Drive the neural CDE's tanh-squashed MLP vector field with this path by converting the ControlTerm to an ODETerm and integrating with diffrax.diffeqsolve using Tsit5 under a diffrax.PIDController with adaptive rtol/atol. Rationale: the thesis finds Hermite cubic splines with backward differences have 'smoothness and relatively good measurability', and neural CDEs using natural cubic splines produce slightly worse results.

## Related tools

- **Diffrax** (Provides backward_hermite_coefficients and CubicInterpolation to build the Hermite cubic control path for the neural CDE) — https://github.com/patrick-kidger/diffrax
- **JAX** (Autodifferentiation framework in which Diffrax is written; enables discretise-then-optimise backpropagation through the interpolated path) — http://github.com/google/jax
- **torchcde** (Prototype PyTorch alternative for CDE interpolation/solving; author recommends Diffrax for new projects) — https://github.com/patrick-kidger/torchcde
- **Equinox** (Defines the tanh-squashed MLP vector field of the neural CDE as an Equinox module on the interpolated path) — https://github.com/patrick-kidger/equinox

## Examples

```
coeffs = diffrax.backward_hermite_coefficients(ts, ys, augmentations=aug); path = diffrax.CubicInterpolation(coeffs)
```

## Evaluation signals

- Spline coefficients from diffrax.backward_hermite_coefficients exactly reproduce the observed values at each observation time (interpolation consistency check).
- The CubicInterpolation path, when driven through the neural CDE on the 2D spiral task, yields 100% test accuracy (test_accuracy.png exists and shows perfect accuracy).
- Each data channel, before interpolation, has zero mean and unit variance.
- Training converges with Tsit5 + PIDController adaptive stepping without solver-failure errors under the default RecursiveCheckpointAdjoint backprop.

## Limitations

- Natural cubic splines are explicitly noted to produce slightly worse neural CDE results, for reasons unknown — avoid them as a default.
- If the Hermite cubic spline's measurability properties are insufficient for the data, rectilinear interpolation is the fallback.
- No JAX library existed at time of writing for computing logsignatures, so logsignature preprocessing alternatives are not available in this stack.
- Optimise-then-discretise backpropagation may be required instead of the default adjoint to avoid running out of memory on long time series.

## Evidence

- [readme] library recommendation: [evidence span withheld — anchor status: not_found]
- [intro] Diffrax-JAX pairing: [evidence span withheld — anchor status: not_found]
