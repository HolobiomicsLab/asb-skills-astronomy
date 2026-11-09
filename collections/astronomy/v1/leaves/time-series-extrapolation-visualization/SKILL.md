---
name: time-series-extrapolation-visualization
description: Use when a generative time-series model (e.g. a latent ODE VAE) trained
  on irregularly-sampled series over a short interval must be evaluated on a much
  longer test horizon, and its samples need to be visualised against held-out trajectories.
when_to_use_negative:
- The model was trained on the same interval it is being evaluated on — no extrapolation
  beyond the training window is being tested, so the four-times-longer rollout and
  its visual comparison are unnecessary.
- Data are regularly sampled grids where discrete RNN/sequence models suffice and
  a continuous-time neural ODE with interpolation-based control path adds no benefit.
- Time series are non-oscillatory or trend-dominated such that continuation beyond
  the training window cannot be meaningfully validated against ground truth.
edam_operation: http://edamontology.org/operation_0438
tools:
- name: Diffrax
  role: Provides diffeqsolve, Tsit5 solver, LinearInterpolation control path, adjoint
    options (RecursiveCheckpointAdjoint / BacksolveAdjoint), and SaveAt checkpointing
    used to roll the latent ODE forward to the extrapolation horizon
  repo: https://github.com/patrick-kidger/diffrax
- name: JAX
  role: Framework in which Diffrax is written; enables autodifferentiable, vmappable
    solving of the latent ODE over the extended interval
  repo: http://github.com/google/jax
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:5d502a7a06af629e90ec42b86ec9ffdecbcbc7a3500d73f725911ca3f16cbc8f
  source_papers:
  - doi: 10.48550/arxiv.2202.02435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/patrick-kidger/diffrax
  tools:
  - Diffrax
  - JAX
  edam_operation: http://edamontology.org/operation_0438
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# time-series-extrapolation-visualization

## Summary

Roll a trained latent ODE far beyond its training time window (e.g. from [0, 3] to [0, 12]) with a JAX/Diffrax neural ODE solver, decode the latent trajectory back to data space, and plot generated sample paths against ground-truth trajectories to assess extrapolation quality.

## When to use

Use when a generative time-series model (e.g. a latent ODE VAE) trained on irregularly-sampled series over a short interval must be evaluated on a much longer test horizon, and its samples need to be visualised against held-out trajectories.

## When NOT to use

- The model was trained on the same interval it is being evaluated on — no extrapolation beyond the training window is being tested, so the four-times-longer rollout and its visual comparison are unnecessary.
- Data are regularly sampled grids where discrete RNN/sequence models suffice and a continuous-time neural ODE with interpolation-based control path adds no benefit.
- Time series are non-oscillatory or trend-dominated such that continuation beyond the training window cannot be meaningfully validated against ground truth.

## Inputs

- Irregularly-sampled decaying-oscillator time series over [0, 3], normalised per channel to zero mean and unit variance
- Trained latent ODE VAE parameters (GRU encoder, neural ODE vector field, Gaussian decoder)
- Test horizon specification: [0, 12] (four times the training interval)

## Outputs

- Extrapolated generated sample paths decoded over [0, 12]
- Overlay plot of generated sample paths versus ground-truth oscillator trajectories
- Saved trained model/solver state (SaveAt-style checkpoint)

## How to apply

Normalise each channel of the irregularly-sampled decaying-oscillator data to zero mean and unit variance, then build the observation times into a continuous control path via linear interpolation (diffrax.LinearInterpolation-style). Train the latent ODE VAE (GRU encoder, neural ODE latent evolution with diffrax.diffeqsolve and the Tsit5 solver, Gaussian decoder head) by maximising the ELBO, backpropagating with the Diffrax default discretise-then-optimise adjoint (RecursiveCheckpointAdjoint), switching to optimise-then-discretise (BacksolveAdjoint) if memory becomes a bottleneck. At evaluation time, call diffeqsolve with t1 set to four times the training horizon (e.g. t1=12 when trained to t=3) and a SaveAt grid spanning the full test interval, decode each saved latent state through the Gaussian head to data space, and overlay the generated sample paths on the ground-truth decaying oscillators. Success is judged visually and by whether the extrapolated samples continue to track the decaying oscillation beyond the training window; save the trained model state alongside the figure.

## Related tools

- **Diffrax** (Provides diffeqsolve, Tsit5 solver, LinearInterpolation control path, adjoint options (RecursiveCheckpointAdjoint / BacksolveAdjoint), and SaveAt checkpointing used to roll the latent ODE forward to the extrapolation horizon) — https://github.com/patrick-kidger/diffrax
- **JAX** (Framework in which Diffrax is written; enables autodifferentiable, vmappable solving of the latent ODE over the extended interval) — http://github.com/google/jax

## Examples

```
sol = diffeqsolve(ODETerm(vector_field), Tsit5(), t0=0, t1=12, dt0=0.1, y0=z0, saveat=SaveAt(ts=jnp.linspace(0, 12, 121))); paths = decode(sol.ys)
```

## Evaluation signals

- Extrapolated decoded samples continue to track the decaying oscillation beyond the training window (i.e. on [3, 12])
- Sample quality over the test interval [0, 12] matches the in-window quality — 'excellent samples' over an interval four times larger than training
- The solver integration covers the full test horizon without step-size or memory failure, with SaveAt capturing states across [0, 12]
- Figure shows generated paths overlaid on ground-truth trajectories, and the trained model state is saved alongside the figure

## Limitations

- Optimise-then-discretise (BacksolveAdjoint) backpropagation may be needed simply to avoid running out of memory on long time series, adding a trade-off between memory and adjoint accuracy.
- The GRU encoder's hidden state involves an exponential-decay term (-h(t)), explaining why GRUs/LSTMs struggle to learn long-term time dependencies — extrapolation relies on the ODE latent dynamics, not the encoder.
- Extrapolation quality beyond the training window is demonstrated for decaying oscillators; generalisation to other dynamics or even longer horizons is not guaranteed.

## Evidence
