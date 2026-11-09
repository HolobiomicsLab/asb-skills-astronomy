---
name: irregular-time-series-interpolation
description: Use when observations arrive at irregular timestamps (e.g. decaying-oscillator
  series sampled non-uniformly over [0,3]) and a continuous path must be constructed
  before solving a neural differential equation.
when_to_use_negative:
- Observations are already on a regular grid — direct tensor input to the model is
  simpler and interpolation adds no timing information.
- The downstream model expects logsignature features over windows (neural CDE preprocessing
  per the thesis) rather than a raw piecewise-linear path.
- Data must stay in raw units for interpretability — normalising each channel to zero
  mean/unit variance changes the value scale and must be inverted at decode time.
tools:
- name: Diffrax
  role: Provides diffrax.LinearInterpolation to turn irregular (ts, ys) observations
    into a piecewise-continuous control path, plus Tsit5 via diffrax.diffeqsolve to
    integrate the downstream latent ODE
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Autodiff framework the interpolated path and solver run under, supporting
    backprop through the solver with RecursiveCheckpointAdjoint or BacksolveAdjoint
  repo: http://github.com/google/jax
- name: torchcde
  role: Prototype PyTorch alternative for building CDE control paths from irregular
    data (Diffrax recommended for new projects)
  repo: https://github.com/patrick-kidger/torchcde
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:ab81044e585add6bf9ce5e6031f6f025681751cc26c0b264a50579ec1cf7230e
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# irregular-time-series-interpolation

## Summary

Builds irregularly-sampled observation times into a continuous control path via piecewise-linear interpolation (diffrax.LinearInterpolation), so that downstream neural ODE/CDE models can consume sequences with non-uniform timestamps.

## When to use

Use when observations arrive at irregular timestamps (e.g. decaying-oscillator series sampled non-uniformly over [0,3]) and a continuous path must be constructed before solving a neural differential equation.

## When NOT to use

- Observations are already on a regular grid — direct tensor input to the model is simpler and interpolation adds no timing information.
- The downstream model expects logsignature features over windows (neural CDE preprocessing per the thesis) rather than a raw piecewise-linear path.
- Data must stay in raw units for interpretability — normalising each channel to zero mean/unit variance changes the value scale and must be inverted at decode time.

## Inputs

- Irregularly-sampled multivariate time series: per-channel observation timestamps ts and observation values ys (e.g. decaying oscillators on [0,3], normalised to zero mean/unit variance)

## Outputs

- A continuous, differentiable control path (diffrax.LinearInterpolation piecewise-linear path) consumable by diffrax.diffeqsolve / neural CDE solvers
- Normalised per-channel data (zero mean, unit variance) feeding the interpolation step

## How to apply

First normalise each channel of the time series to zero mean and unit variance, matching the thesis's preprocessing convention. Then wrap the irregular observations (timestamps ts, values ys) in a diffrax.LinearInterpolation path — a piecewise-continuous path that is CDE-compatible — rather than resampling onto a regular grid, which would discard the timing information. The interpolation scheme is a decision point: Hermite cubic splines with backward differences are recommended when smoothness is needed, but rectilinear/linear interpolation is the fallback when measurability properties are insufficient (natural cubic splines produced slightly worse results). If interpolation is feeding a neural CDE rather than an encoder-based latent ODE, the thesis additionally preprocesses windows into logsignatures. Mind memory: discretise-then-optimise adjoint (RecursiveCheckpointAdjoint) is the default; switch to optimise-then-discretise (BacksolveAdjoint) when long interpolated sequences exhaust memory.

## Related tools

- **Diffrax** (Provides diffrax.LinearInterpolation to turn irregular (ts, ys) observations into a piecewise-continuous control path, plus Tsit5 via diffrax.diffeqsolve to integrate the downstream latent ODE) — https://github.com/patrick-kidger/diffrax
- **JAX** (Autodiff framework the interpolated path and solver run under, supporting backprop through the solver with RecursiveCheckpointAdjoint or BacksolveAdjoint) — http://github.com/google/jax
- **torchcde** (Prototype PyTorch alternative for building CDE control paths from irregular data (Diffrax recommended for new projects)) — https://github.com/patrick-kidger/torchcde

## Examples

```
import diffrax; path = diffrax.LinearInterpolation(ts=obs_ts, ys=obs_ys)
```

## Evaluation signals

- Per-channel statistics of the normalised series have mean ≈ 0 and variance ≈ 1.
- The interpolated path is piecewise-linear and continuous: value at any observation time ts_i equals the observed ys_i exactly.
- Downstream latent ODE solved on the path reproduces the decaying oscillation over the training window [0,3] and, at test time, continues to track the decaying oscillation on [0,12] (4× extrapolation).
- Decoded sample paths plotted against ground-truth oscillator trajectories visually align beyond the training window; trained model state and figure are saved.

## Limitations

- Natural cubic splines produced slightly worse results than other interpolation schemes; prefer Hermite cubic splines with backward differences, falling back to rectilinear when measurability is insufficient.
- Optimise-then-discretise (BacksolveAdjoint) may be needed simply to avoid running out of memory on long interpolated sequences.
- At the time of writing no JAX library exists for computing logsignatures, limiting the CDE logsignature-preprocessing variant in the JAX ecosystem.

## Evidence
