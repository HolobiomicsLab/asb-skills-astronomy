---
name: latent-ode-variational-inference
description: Use when you have irregularly-sampled continuous-time series, need generative
  modelling of latent dynamics, and must extrapolate far beyond the training window
  (e.g. train on [0, 3], test on [0, 12]).
when_to_use_negative:
- The task requires controlled (input-driven) dynamics rather than autonomous latent
  evolution — a neural CDE with an interpolated control path, not a latent ODE, is
  the appropriate model
- 'Long-term time dependencies must be learned directly by a recurrent encoder: the
  thesis notes the -h(t) term in the continuous-time GRU causes exponential decay
  of the hidden state, explaining why GRUs/LSTMs struggle to learn long-term dependencies'
- Only short-horizon interpolation within the training window is needed — the variational
  latent ODE machinery is aimed at generative extrapolation beyond the training interval
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3474
tools:
- name: Diffrax
  role: 'ODE solver library: diffrax.diffeqsolve with Tsit5 solver, LinearInterpolation
    control path, adjoint options (RecursiveCheckpointAdjoint / BacksolveAdjoint),
    SaveAt state'
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Autodifferentiation framework in which Diffrax, Equinox and Optax are composed
    for the VAE training loop
  repo: http://github.com/google/jax
- name: Equinox
  role: Neural network / model parameterisation (PyTree modules) used with JAX and
    Diffrax
  repo: https://github.com/patrick-kidger/equinox
- name: Optax
  role: Gradient processing / optimisation library for the JAX-based ELBO training
    loop
  repo: https://github.com/google-deepmind/optax
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:18d3cbc63602d9dd0ff5bf84b28bbeb3e5f6d112f56d5d77381f024be10367c2
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Latent ODE variational inference

## Summary

Train a latent ODE VAE (GRU encoder, neural ODE latent dynamics solved with diffrax Tsit5, Gaussian decoder) on irregularly-sampled decaying-oscillator time series over [0, 3] by maximising the ELBO, so the trained model can be rolled forward to generate samples over a four-times-longer horizon [0, 12].

## When to use

Use when you have irregularly-sampled continuous-time series, need generative modelling of latent dynamics, and must extrapolate far beyond the training window (e.g. train on [0, 3], test on [0, 12]).

## When NOT to use

- The task requires controlled (input-driven) dynamics rather than autonomous latent evolution — a neural CDE with an interpolated control path, not a latent ODE, is the appropriate model
- Long-term time dependencies must be learned directly by a recurrent encoder: the thesis notes the -h(t) term in the continuous-time GRU causes exponential decay of the hidden state, explaining why GRUs/LSTMs struggle to learn long-term dependencies
- Only short-horizon interpolation within the training window is needed — the variational latent ODE machinery is aimed at generative extrapolation beyond the training interval

## Inputs

- Irregularly-sampled decaying-oscillator time series over [0, 3] (synthetic, per thesis specification)
- Per-channel observation times and values, normalised to zero mean and unit variance

## Outputs

- Trained latent ODE VAE model state (e.g. saved via SaveAt-style solver/parameter state)
- Generated sample paths decoded over the extrapolation interval [0, 12]
- Plot of generated sample paths against ground-truth oscillator trajectories

## How to apply

Generate or take the decaying-oscillator time series with irregular observation times and normalise each channel to zero mean and unit variance. Build the observations into a continuous control path via linear interpolation (diffrax.LinearInterpolation-style piecewise-continuous path). Construct the latent ODE VAE: a GRU encoder maps the observed sequence to the initial latent state, a neural ODE evolves the latent state solved with diffrax.diffeqsolve using the Tsit5 solver, and a Gaussian head decodes latent states back to observations. Train by maximising the ELBO, backpropagating through the solver with the Diffrax default discretise-then-optimise adjoint (RecursiveCheckpointAdjoint), switching to optimise-then-discretise (BacksolveAdjoint) if memory becomes a bottleneck on long sequences. Finally roll the trained latent ODE forward over the four-times-longer test horizon, decode to data space, and compare generated sample paths against ground-truth oscillator trajectories.

## Related tools

- **Diffrax** (ODE solver library: diffrax.diffeqsolve with Tsit5 solver, LinearInterpolation control path, adjoint options (RecursiveCheckpointAdjoint / BacksolveAdjoint), SaveAt state) — https://github.com/patrick-kidger/diffrax
- **JAX** (Autodifferentiation framework in which Diffrax, Equinox and Optax are composed for the VAE training loop) — http://github.com/google/jax
- **Equinox** (Neural network / model parameterisation (PyTree modules) used with JAX and Diffrax) — https://github.com/patrick-kidger/equinox
- **Optax** (Gradient processing / optimisation library for the JAX-based ELBO training loop) — https://github.com/google-deepmind/optax

## Examples

```
sol = diffrax.diffeqsolve(diffrax.ODETerm(f), diffrax.Tsit5(), t0=0, t1=12, dt0=None, y0=z0, saveat=diffrax.SaveAt(ts=ts))
```

## Evaluation signals

- By the end of training the extrapolated samples over [0, 12] continue to track the decaying oscillation beyond the training window (four times the training interval)
- Each input channel is normalised to zero mean and unit variance before modelling
- The ELBO loss decreases during training and backpropagation through diffeqsolve succeeds without memory blow-up; switch to BacksolveAdjoint if it does
- Saved model state (SaveAt-style solver/parameter state) can be reloaded and re-decoded to reproduce the sample-path figure

## Limitations

- Backpropagation through the solver with the default discretise-then-optimise adjoint can exhaust memory on long time series — optimise-then-discretise was needed in the thesis simply to avoid running out of memory
- Neural CDEs using natural cubic splines produced slightly worse results than other interpolation schemes; Hermite cubic splines with backward differences are recommended when smoothness/measurability permit, otherwise rectilinear
- Extrapolation quality is demonstrated only for the decaying-oscillator synthetic task over a 4x horizon; generalisation to other dynamics is unverified

## Evidence

- [discussion] By the end of training, the latent ODE produces excellent samples over the test interval [0, 12], i.e. a time interval four times larger than the interval it was trained on, demonstrating successful extrapolation.: [evidence span withheld — anchor status: not_found]
- [discussion] Each channel is normalised to zero mean and unit variance.: [evidence span withheld — anchor status: not_found]
