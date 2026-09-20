---
name: synthetic-oscillator-data-generation
description: Use when you need synthetic irregularly-sampled time series with known
  ground-truth dynamics (e.g. to train or benchmark latent ODEs or extrapolation-capable
  neural differential equation models) and want normalised, controllable data per
  the thesis specification.
when_to_use_negative:
- The task uses real-world irregularly-sampled datasets (e.g. the BIDMC physiological
  time series used in the thesis experiments) rather than synthetic ground-truth dynamics
- The downstream model requires a JAX logsignature-based control path (e.g. neural
  CDE preprocessing), since 'at time of writing there does not exist a JAX library
  for computing logsignatures' and this skill produces linearly-interpolable observations
  instead
tools:
- name: JAX
  role: Framework in which the synthetic time series generation, normalisation, and
    the downstream Diffrax-based latent ODE are implemented
  repo: http://github.com/google/jax
- name: Diffrax
  role: JAX-based differential equation solver library used to consume the generated
    data (diffeqsolve with Tsit5) and available as reference example implementations
  repo: https://github.com/patrick-kidger/diffrax
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:1c578758767bfbabc779476a758486b0490770d4ace25ab1bf48646f84000d60
  source_papers:
  - doi: 10.48550/arxiv.2202.02435
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Diffrax
  - JAX
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# synthetic-oscillator-data-generation

## Summary

Generates synthetic decaying-oscillator time series sampled at irregular observation times over [0, 3], with per-channel normalisation to zero mean and unit variance, for use as training data in latent ODE / neural ODE models built on Diffrax and JAX.

## When to use

Use when you need synthetic irregularly-sampled time series with known ground-truth dynamics (e.g. to train or benchmark latent ODEs or extrapolation-capable neural differential equation models) and want normalised, controllable data per the thesis specification.

## When NOT to use

- The task uses real-world irregularly-sampled datasets (e.g. the BIDMC physiological time series used in the thesis experiments) rather than synthetic ground-truth dynamics
- The downstream model requires a JAX logsignature-based control path (e.g. neural CDE preprocessing), since 'at time of writing there does not exist a JAX library for computing logsignatures' and this skill produces linearly-interpolable observations instead

## Inputs

- Thesis specification of the decaying-oscillator system (decay rates, frequencies, noise, sampling process)
- RNG seed / number of trajectories and channels
- Training interval [0, 3] and extrapolation test interval [0, 12]

## Outputs

- Irregularly-sampled decaying-oscillator observation sequences (times + values per channel)
- Per-channel normalisation statistics (zero mean, unit variance)
- Ground-truth continuous trajectories over [0, 3] and [0, 12] for validation and plotting

## How to apply

Generate the synthetic decaying-oscillator time series with irregular observation times per the thesis specification: sample observation times irregularly over the training interval [0, 3] and compute the decaying-oscillator trajectories at those times. Apply the thesis preprocessing rule that each channel is normalised to have zero mean and unit variance, computing statistics per channel over the dataset. Retain the ground-truth continuous trajectories and observation-time metadata so a downstream model (e.g. the latent ODE VAE with GRU encoder, diffrax.diffeqsolve Tsit5 latent evolution, and Gaussian decoder) can build a continuous control path via linear interpolation and be evaluated on extrapolation to [0, 12], four times the training horizon. Because the data is synthetic, exact ground truth is available at every test time point, which is what makes quantitative extrapolation validation possible.

## Related tools

- **JAX** (Framework in which the synthetic time series generation, normalisation, and the downstream Diffrax-based latent ODE are implemented) — http://github.com/google/jax
- **Diffrax** (JAX-based differential equation solver library used to consume the generated data (diffeqsolve with Tsit5) and available as reference example implementations) — https://github.com/patrick-kidger/diffrax

## Examples

```
import diffrax; sol = diffrax.diffeqsolve(diffrax.ODETerm(vector_field), diffrax.Tsit5(), t0=0.0, t1=12.0, dt0=0.1, y0=z0, saveat=diffrax.SaveAt(ts=obs_times))
```

## Evaluation signals

- Each output channel has mean 0 and standard deviation 1 (per-channel normalisation check)
- Observation timestamps are strictly increasing and irregular (non-uniform spacing) within [0, 3]
- Generated trajectories exhibit decaying oscillation amplitude consistent with the thesis system, verified against the analytic ground truth
- A held-out ground-truth trajectory over [0, 12] exists so extrapolated model samples can be plotted against truth beyond the training window

## Limitations

- The training interval is fixed at [0, 3] with the test/extrapolation interval at [0, 12]; other horizons require regenerating data and re-validating extrapolation claims
- Synthetic decaying oscillators are smooth and low-dimensional; performance on this data does not guarantee transfer to real-world irregular time series such as the BIDMC datasets
- Normalisation is per channel; channels whose dynamics are nearly flat may become noise-dominated after unit-variance scaling

## Evidence

- [discussion] Latent ODE extrapolation finding motivating the synthetic data: [evidence span withheld — anchor status: not_found]
