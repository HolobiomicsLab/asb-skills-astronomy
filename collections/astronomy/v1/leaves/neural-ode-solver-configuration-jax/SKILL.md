---
name: neural-ode-solver-configuration-jax
description: Use when irregularly-sampled time series must be modelled with a latent
  ODE and the goal is to generate or forecast samples beyond the training time window
  (e.g. training on [0,3], testing on [0,12]).
when_to_use_negative:
- The data is a fixed, regularly sampled grid with no irregular observation times
  and no need for extrapolation beyond the training window — a standard sequence model
  suffices
- The system of interest is a controlled/stochastic process best expressed as a neural
  CDE or neural SDE (the thesis treats these separately, e.g. neural SDEs solved with
  the reversible Heun method)
- Natural cubic spline interpolation is required for measurability — the thesis reports
  neural CDEs using natural cubic splines 'produce slightly worse results than those
  using other interpolation schemes'
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3316
tools:
- name: Diffrax
  role: Provides diffeqsolve, the Tsit5 solver, LinearInterpolation-style control
    paths, adjoint options (RecursiveCheckpointAdjoint / BacksolveAdjoint), and SaveAt
    state saving for the latent ODE
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Framework Diffrax is written for; supplies autodiff and vmapping needed for
    ELBO training of the latent ODE VAE
  repo: http://github.com/google/jax
- name: Equinox
  role: Used with JAX, Diffrax, and Optax to implement the model (neural network modules
    for encoder/decoder/vector field)
  repo: https://github.com/patrick-kidger/equinox
- name: Optax
  role: Gradient-processing/optimisation library used alongside JAX and Diffrax in
    the thesis implementation
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:095261eea8e25077670826be2da746558aea12e8c497e8ef61524b0ad1300859
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
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# neural-ode-solver-configuration-jax

## Summary

Configure and train a latent ODE in JAX/Diffrax (GRU encoder, neural ODE latent dynamics via diffeqsolve with Tsit5, Gaussian decoder) and validate its extrapolation to horizons longer than the training interval, e.g. decaying-oscillator series trained on [0,3] and sampled on [0,12].

## When to use

Use when irregularly-sampled time series must be modelled with a latent ODE and the goal is to generate or forecast samples beyond the training time window (e.g. training on [0,3], testing on [0,12]).

## When NOT to use

- The data is a fixed, regularly sampled grid with no irregular observation times and no need for extrapolation beyond the training window — a standard sequence model suffices
- The system of interest is a controlled/stochastic process best expressed as a neural CDE or neural SDE (the thesis treats these separately, e.g. neural SDEs solved with the reversible Heun method)
- Natural cubic spline interpolation is required for measurability — the thesis reports neural CDEs using natural cubic splines 'produce slightly worse results than those using other interpolation schemes'

## Inputs

- Irregularly-sampled decaying-oscillator time series over [0, 3] (synthetic, per thesis specification)
- Per-channel normalisation statistics (zero mean, unit variance per channel)
- Observation timestamps for building the piecewise-linear control path

## Outputs

- Trained latent ODE VAE model state saved via SaveAt-style solver/parameter state
- Generated sample paths decoded over the extrapolated test interval [0, 12]
- Figure plotting generated sample paths against ground-truth oscillator trajectories

## How to apply

Generate the synthetic decaying-oscillator series with irregular observation times and normalise each channel to zero mean and unit variance. Build the observations into a continuous control path via linear interpolation (diffrax.LinearInterpolation-style piecewise-continuous path). Build a latent ODE VAE: a GRU encoder maps the observed sequence to the initial latent state, the latent state evolves under a neural ODE solved with diffrax.diffeqsolve using the Tsit5 solver, and a Gaussian head decodes latent states back to data space. Train by maximising the ELBO, backpropagating through the solver with Diffrax's default discretise-then-optimise adjoint (RecursiveCheckpointAdjoint), switching to optimise-then-discretise (BacksolveAdjoint) if memory becomes a bottleneck on long sequences — the thesis notes optimise-then-discretise 'was needed simply to avoid running out of memory'. Finally solve the trained latent ODE over the four-times-longer test horizon, decode, and plot generated paths against ground truth to confirm the decaying oscillation is tracked beyond the training window; save the model state via SaveAt-style solver/parameter state.

## Related tools

- **Diffrax** (Provides diffeqsolve, the Tsit5 solver, LinearInterpolation-style control paths, adjoint options (RecursiveCheckpointAdjoint / BacksolveAdjoint), and SaveAt state saving for the latent ODE) — https://github.com/patrick-kidger/diffrax
- **JAX** (Framework Diffrax is written for; supplies autodiff and vmapping needed for ELBO training of the latent ODE VAE) — http://github.com/google/jax
- **Equinox** (Used with JAX, Diffrax, and Optax to implement the model (neural network modules for encoder/decoder/vector field)) — https://github.com/patrick-kidger/equinox
- **Optax** (Gradient-processing/optimisation library used alongside JAX and Diffrax in the thesis implementation)

## Examples

```
term = diffrax.ODETerm(vector_field); sol = diffrax.diffeqsolve(term, diffrax.Tsit5(), t0=0, t1=12, dt0=0.1, y0=z0, saveat=diffrax.SaveAt(ts=ts_test))
```

## Evaluation signals

- Extrapolated samples over [0, 12] continue to track the decaying oscillation beyond the training window [0, 3] when plotted against ground truth
- The trained model produces 'excellent samples' over the test interval, i.e. a time interval four times larger than the training interval
- Each data channel has zero mean and unit variance after preprocessing
- Training loss (negative ELBO) decreases monotonically with the chosen adjoint; no out-of-memory failures on long sequences (switch to BacksolveAdjoint if they occur)
- Saved model state (SaveAt-style solver/parameter state) reloads and reproduces the extrapolated sample paths

## Limitations

- Backpropagation memory: with the default discretise-then-optmise adjoint, long time series may run out of memory; the thesis reports optimise-then-discretise 'was needed simply to avoid running out of memory on long time series'
- At time of writing there did not exist a JAX library for computing logsignatures, restricting some CDE-style preprocessing options in the JAX ecosystem
- The article notes no JAX library for logsignatures and that higher order reversible ODE solvers / hypersolvers during training were not yet available
- Extrapolation success was demonstrated on synthetic decaying oscillators; no software version numbers are stated for any tool

## Evidence

- [discussion] Latent ODE training and adjoint choice: [evidence span withheld — anchor status: not_found]
- [discussion] Diffrax examples: [evidence span withheld — anchor status: not_found]
- [intro] Diffrax is a JAX library: [evidence span withheld — anchor status: not_found]
