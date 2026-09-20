---
name: monte-carlo-standard-error-analysis
description: Use when MCMC chains (e.g. emcee ensemble walkers or CosmoMC Metropolis–Hastings
  chains) must be shown to lose their initialisation bias before posterior samples
  are trusted, i.e. to fix a defensible burn-in length.
when_to_use_negative:
- The goal is wall-time/parallel-scaling benchmarking of the sampler rather than convergence/burn-in
  determination
- The chains come from a single-walker sampler where the ensemble mean vs sigma/sqrt(L)
  criterion does not apply
- Samples are needed only after the CosmoMC proposal-tuning phase (~2000 steps) has
  already been run — then the tuning phase, not the burn-in analysis, is the limiting
  step
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CosmoHammer
  role: Python framework that runs the emcee ensemble sampling of WMAP7 cosmological
    parameters, producing the walker chains analysed for burn-in
  repo: https://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Implementation of the Goodman–Weare affine invariant ensemble sampler embedded
    in CosmoHammer; its walker ensemble supplies X̄_t and L for the sigma/sqrt(L)
    comparison
  repo: http://arxiv.org/abs/1202.3665
- name: CosmoMC
  role: Metropolis–Hastings comparison sampler whose chains require proposal tuning
    (~2000 steps) and whose burn-in is determined with the same standard-error criterion
- name: acor
  role: Python port of Goodman's ACOR that estimates the autocorrelation time tau
    of the chain time series, relevant to why burn-in lengths differ across samplers
  repo: https://github.com/dfm/acor
- name: Python
  role: Language in which CosmoHammer and the convergence monitoring scripts are implemented
  repo: https://github.com/cosmo-ethz/CosmoHammer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:83c35b8efadf8a5b296fb61a174827537fcc76088255519939e8c9dc7572ac0a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cosmo-ethz/CosmoHammer
  repo_url: https://github.com/cosmo-ethz/CosmoHammer
  tools:
  - CosmoHammer
  - CosmoMC
  - Python
  - acor
  - emcee
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# monte-carlo-standard-error-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Determines a safe MCMC burn-in length by tracking the ensemble mean sequential step X̄_t of walkers/chains against the predicted standard error of the mean sigma/sqrt(L), and comparing autocorrelation behaviour of ensemble (emcee) and Metropolis–Hastings samplers.

## When to use

Use when MCMC chains (e.g. emcee ensemble walkers or CosmoMC Metropolis–Hastings chains) must be shown to lose their initialisation bias before posterior samples are trusted, i.e. to fix a defensible burn-in length.

## When NOT to use

- The goal is wall-time/parallel-scaling benchmarking of the sampler rather than convergence/burn-in determination
- The chains come from a single-walker sampler where the ensemble mean vs sigma/sqrt(L) criterion does not apply
- Samples are needed only after the CosmoMC proposal-tuning phase (~2000 steps) has already been run — then the tuning phase, not the burn-in analysis, is the limiting step

## Inputs

- Posterior sample chains from CosmoHammer (emcee ensemble sampler) for the six WMAP7 cosmological parameters (e.g. walkers ratio 50 → 300 walkers)
- Posterior chains from fine-tuned/self-tuning CosmoMC Metropolis–Hastings runs (including ~2000-step proposal-tuning phase)
- Number of walkers/chains L and per-parameter sample sigma

## Outputs

- Burn-in length determination per sampler configuration (e.g. 250 steps for both CosmoHammer and fine-tuned CosmoMC; ~2000 tuning steps for self-tuning CosmoMC)
- Plot of ensemble mean sequential step X̄_t vs iteration with the sigma/sqrt(L) band (e.g. burnin_convergence_analysis.png)
- Burn-in lengths table (e.g. burnin_summary.csv)
- Optional autocorrelation time tau per chain from acor

## How to apply

Load the posterior samples from the chains (e.g. the six WMAP7 cosmological parameters sampled by CosmoHammer/emcee, where a walkers ratio of 50 launches 300 walkers). Compute the ensemble mean sequential step X̄_t as a function of iteration t across all walkers, and estimate the scatter of X̄_t across realisations. Compare this scatter with the predicted standard error of the mean sigma/sqrt(L), where L is the number of walkers (or chains); the initialisation bias vanishes once X̄_t fluctuates within the sigma/sqrt(L) band. Repeat the tracking for the comparison sampler — self-tuning CosmoMC requires a roughly 2000-step proposal-tuning phase before its samples become usable — and record the burn-in lengths (250 steps was found to be safe for both CosmoHammer configurations despite different autocorrelation times). Autocorrelation times can be quantified with the acor routine, a Python port of Goodman's ACOR that returns tau, mean and sigma for a time series. Plot the X̄_t trajectories with the sigma/sqrt(L) band and export the burn-in summary for validation against reported values.

## Related tools

- **CosmoHammer** (Python framework that runs the emcee ensemble sampling of WMAP7 cosmological parameters, producing the walker chains analysed for burn-in) — https://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Implementation of the Goodman–Weare affine invariant ensemble sampler embedded in CosmoHammer; its walker ensemble supplies X̄_t and L for the sigma/sqrt(L) comparison) — http://arxiv.org/abs/1202.3665
- **CosmoMC** (Metropolis–Hastings comparison sampler whose chains require proposal tuning (~2000 steps) and whose burn-in is determined with the same standard-error criterion)
- **acor** (Python port of Goodman's ACOR that estimates the autocorrelation time tau of the chain time series, relevant to why burn-in lengths differ across samplers) — https://github.com/dfm/acor
- **Python** (Language in which CosmoHammer and the convergence monitoring scripts are implemented) — https://github.com/cosmo-ethz/CosmoHammer

## Examples

```
import acor
for walker_chain in chains:
    tau, mean, sigma = acor.acor(walker_chain)
```

## Evaluation signals

- The determined burn-in length equals the paper's reported value of 250 steps for both CosmoHammer configurations and fine-tuned CosmoMC
- Self-tuning CosmoMC chains show approximately 2000 steps of proposal tuning before samples become usable
- After burn-in, the ensemble mean sequential step X̄_t remains within the sigma/sqrt(L) band across realisations
- The walkers ratio of 50 yields L = 300 walkers for six parameters, matching the paper's configuration
- The summary table and trajectory figure (e.g. burnin_summary.csv, burnin_convergence_analysis.png) are produced and consistent with each other

## Limitations

- Scaling saturates once more processes are available than walkers: processes idle, so the analysis applies to chain content, not to parallel speed-up
- Self-tuning CosmoMC needs a ~2000-step tuning phase before its samples become usable, so identical burn-in does not imply identical effective cost across samplers
- The analysis was demonstrated on six WMAP7 cosmological parameters; burn-in length may differ for other posteriors or autocorrelation structures

## Evidence

- [intro] it is also a good reference point for the performance of the algorithm by Goodman and Weare as compared to a Metropolis–Hastings sampler: [evidence span withheld — anchor status: not_found]
- [readme] estimates the autocorrelation time of time series data very quickly: [evidence span withheld — anchor status: external_doc]
