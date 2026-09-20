---
name: cross-sampler-convergence-comparison
description: Use when parallelised MCMC chains from different samplers (e.g. emcee
  ensemble walkers vs. Metropolis–Hastings chains) must be compared for initialisation
  bias and a burn-in length must be justified numerically.
when_to_use_negative:
- Only one sampler is being run and no cross-sampler comparison is needed — a single-chain
  convergence diagnostic suffices and the X̄_t vs sigma/sqrt(L) comparison adds no
  information.
- The likelihood is not the WMAP7 CMB posterior with six sampled cosmological parameters
  — the walkers ratio of 50 (walkers = 50 × n_parameters) and the calibrated 250-step
  burn-in were determined for that specific posterior and must be re-derived for other
  problems.
- The CosmoMC chains are still within their ~2000-step self-tuning/proposal-adaptation
  phase — such samples are not yet usable and would contaminate the burn-in estimate.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: CosmoHammer
  role: Framework embedding emcee that generates the ensemble MCMC chains whose burn-in
    is tracked and compared
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Affine-invariant Goodman and Weare ensemble sampler producing the L-walker
    chains used to compute X̄_t
  repo: http://arxiv.org/abs/1202.3665
- name: CosmoMC
  role: Metropolis–Hastings reference sampler (with CAMB) whose self-tuning chains
    provide the comparison arm of the convergence analysis
- name: acor
  role: Python module estimating the autocorrelation time tau of the time-series chains,
    which differ between the samplers
  repo: https://github.com/dfm/acor
- name: Python
  role: Language in which the X̄_t trajectories, sigma/sqrt(L) band, and summary outputs
    are computed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:81128500ba67117cf0136ef8d6ef782f46e82b5f4e7546a4e819805dd8b2604b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cosmo-ethz/CosmoHammer
  tools:
  - CosmoHammer
  - CosmoMC
  - Python
  - acor
  - emcee
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Cross-sampler convergence comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A protocol for comparing burn-in / convergence behaviour of two MCMC samplers (CosmoHammer's emcee ensemble sampler vs. the Metropolis–Hastings sampler of fine-tuned CosmoMC) by tracking the ensemble mean sequential step X̄_t against the predicted standard error sigma/sqrt(L), yielding a common burn-in length of 250 iterations despite different autocorrelation times.

## When to use

Use when parallelised MCMC chains from different samplers (e.g. emcee ensemble walkers vs. Metropolis–Hastings chains) must be compared for initialisation bias and a burn-in length must be justified numerically.

## When NOT to use

- Only one sampler is being run and no cross-sampler comparison is needed — a single-chain convergence diagnostic suffices and the X̄_t vs sigma/sqrt(L) comparison adds no information.
- The likelihood is not the WMAP7 CMB posterior with six sampled cosmological parameters — the walkers ratio of 50 (walkers = 50 × n_parameters) and the calibrated 250-step burn-in were determined for that specific posterior and must be re-derived for other problems.
- The CosmoMC chains are still within their ~2000-step self-tuning/proposal-adaptation phase — such samples are not yet usable and would contaminate the burn-in estimate.

## Inputs

- Posterior sample chains from CosmoHammer (emcee) runs on the six WMAP7 cosmological parameters (walkers ratio 50 → 300 walkers)
- Posterior sample chains from self-tuning CosmoMC Metropolis–Hastings runs (including ~2000-step proposal-tuning phase)
- Time series of walker/chain sequential steps per iteration t

## Outputs

- burnin_convergence_analysis.png — plot of X̄_t trajectories with the sigma/sqrt(L) band
- burnin_summary.csv — table of determined burn-in lengths per sampler (250 steps for both configurations; ~2000 tuning steps for CosmoMC)
- Autocorrelation time estimates (tau) per sampler, e.g. via acor

## How to apply

Load the posterior samples from the CosmoHammer (emcee) chains for the six WMAP7 cosmological parameters — a walkers ratio of 50 launches 50 × 6 = 300 walkers — and compute the ensemble mean sequential step X̄_t as a function of iteration t across all walkers. Estimate the scatter of X̄_t across realisations and compare it with the predicted standard error sigma/sqrt(L), where L is the number of walkers/chains; the iteration at which X̄_t first fluctuates within the sigma/sqrt(L) band marks where initialisation bias vanishes. Repeat the identical tracking for the self-tuning CosmoMC Metropolis–Hastings chains, which need a roughly 2000-step proposal-tuning phase before samples become usable. Record the burn-in lengths per sampler (a safe burn-in of 250 steps for both CosmoHammer configurations) and plot X̄_t trajectories with the sigma/sqrt(L) band. Autocorrelation times, which differ between samplers and justify the identical burn-in, can be estimated with the acor Python module (a port of Goodman's ACOR routine returning tau, mean, sigma).

## Related tools

- **CosmoHammer** (Framework embedding emcee that generates the ensemble MCMC chains whose burn-in is tracked and compared) — http://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Affine-invariant Goodman and Weare ensemble sampler producing the L-walker chains used to compute X̄_t) — http://arxiv.org/abs/1202.3665
- **CosmoMC** (Metropolis–Hastings reference sampler (with CAMB) whose self-tuning chains provide the comparison arm of the convergence analysis)
- **acor** (Python module estimating the autocorrelation time tau of the time-series chains, which differ between the samplers) — https://github.com/dfm/acor
- **Python** (Language in which the X̄_t trajectories, sigma/sqrt(L) band, and summary outputs are computed)

## Examples

```
import acor; tau, mean, sigma = acor.acor(x)
```

## Evaluation signals

- The determined burn-in length reproduces the paper's reported value of 250 steps for both the CosmoHammer and fine-tuned CosmoMC configurations.
- CosmoMC's proposal-tuning phase is recorded as approximately 2000 steps before its samples become usable.
- After burn-in, X̄_t remains within the sigma/sqrt(L) band across realisations.
- The walkers ratio setting is consistent with the paper: ratio 50 with six parameters launches 300 walkers.
- burnin_summary.csv contains one burn-in entry per sampler and burnin_convergence_analysis.png shows trajectories with the predicted standard-error band.

## Limitations

- Scaling saturates: once more processes than walkers are available, processes idle (plateau at 64 nodes), so L cannot be scaled arbitrarily to shrink sigma/sqrt(L).
- The current CosmoHammer distribution was only tested with Intel's ifort compiler and mkl libraries; other Fortran compilers are untested, which may affect reproducibility of the CosmoMC comparison arm.
- The 11% scaling overhead when doubling cores (α = 0.89) means wall-time gains are not perfectly linear, which affects how much parallelised burn-in can shorten wall time.

## Evidence

- [discussion] emcee chosen for its parallelisation preconditions: [evidence span withheld — anchor status: not_found]
