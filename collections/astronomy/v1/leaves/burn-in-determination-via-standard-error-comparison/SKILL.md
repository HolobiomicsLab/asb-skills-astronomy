---
name: burn-in-determination-via-standard-error-comparison
description: Use when parallelised ensemble MCMC chains (e.g. emcee walkers or Metropolis-Hastings
  chains) must be checked for residual initialisation bias, and a defensible burn-in
  length must be derived rather than assumed.
when_to_use_negative:
- The analysis goal is wall-time/scaling performance of the sampler (core-count sweeps),
  not chain convergence or burn-in length.
- The chains are already demonstrably converged and only posterior summaries are needed;
  the burn-in comparison adds no information.
- The input is a single short chain where the ensemble-mean vs sigma/sqrt(L) comparison
  over many walkers/chains cannot be estimated reliably.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: CosmoHammer
  role: Python framework embedding emcee that generates the parallelised MCMC walker
    chains whose burn-in is being determined
  repo: https://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Affine invariant ensemble sampler (Goodman and Weare) whose walker trajectories
    are monitored via the mean sequential step
- name: CosmoMC
  role: Self-tuning Metropolis-Hastings sampler used as a comparison; requires a ~2000-step
    proposal-tuning phase before burn-in tracking
- name: acor
  role: Python tool for estimating the autocorrelation time of time series, relevant
    to interpreting walker convergence
  repo: https://github.com/dfm/acor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:2e38ad71eea5a3bfe55273a0fb800e70c3fb5278bc0e74825041ed3770f93cf4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# burn-in-determination-via-standard-error-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Determines a safe MCMC burn-in length by monitoring the ensemble mean sequential step of walkers (X̄_t) against the predicted standard error of the mean, sigma/sqrt(L), as done for Cosmo Hammer and fine-tuned CosmoMC on WMAP7 cosmological parameters.

## When to use

Use when parallelised ensemble MCMC chains (e.g. emcee walkers or Metropolis-Hastings chains) must be checked for residual initialisation bias, and a defensible burn-in length must be derived rather than assumed.

## When NOT to use

- The analysis goal is wall-time/scaling performance of the sampler (core-count sweeps), not chain convergence or burn-in length.
- The chains are already demonstrably converged and only posterior summaries are needed; the burn-in comparison adds no information.
- The input is a single short chain where the ensemble-mean vs sigma/sqrt(L) comparison over many walkers/chains cannot be estimated reliably.

## Inputs

- Posterior sample chains from CosmoHammer (emcee) runs, e.g. 300 walkers for the six WMAP7 cosmological parameters
- Posterior sample chains from self-tuning CosmoMC Metropolis-Hastings runs
- Number of walkers/chains L and posterior scatter sigma for the sampled parameters

## Outputs

- Trajectory figure of the ensemble mean sequential step X̄_t with the sigma/sqrt(L) band (burnin_convergence_analysis.png)
- Table of determined burn-in lengths (burnin_summary.csv, e.g. 250 steps for both configurations, ~2000 tuning steps for CosmoMC)

## How to apply

Load the posterior samples from the emcee/CosmoHammer chains (e.g. 300 walkers launched from a walkers ratio of 50 for the six WMAP7 cosmological parameters) and compute the ensemble mean sequential step X̄_t as a function of iteration t across all walkers. Estimate the scatter of X̄_t across realisations and compare it with the predicted standard error of the mean, sigma/sqrt(L), where L is the number of walkers (for ensemble samplers) or chains (for Metropolis-Hastings). The iteration at which X̄_t first fluctuates within the sigma/sqrt(L) band marks where the initialisation bias has vanished; the rationale is that once walkers sample the target distribution, the ensemble mean behaves like a simple random sample with standard error sigma/sqrt(L). Repeat the same tracking for self-tuning CosmoMC Metropolis-Hastings chains, which first require a roughly 2000-step proposal-tuning phase before samples are usable. Record the resulting burn-in lengths (a safe value of 250 iterations for both Cosmo Hammer and fine-tuned CosmoMC) and save the X̄_t trajectories with the sigma/sqrt(L) band plus a burn-in summary table.

## Related tools

- **CosmoHammer** (Python framework embedding emcee that generates the parallelised MCMC walker chains whose burn-in is being determined) — https://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Affine invariant ensemble sampler (Goodman and Weare) whose walker trajectories are monitored via the mean sequential step)
- **CosmoMC** (Self-tuning Metropolis-Hastings sampler used as a comparison; requires a ~2000-step proposal-tuning phase before burn-in tracking)
- **acor** (Python tool for estimating the autocorrelation time of time series, relevant to interpreting walker convergence) — https://github.com/dfm/acor

## Examples

```
import acor; tau, mean, sigma = acor.acor(x)
```

## Evaluation signals

- The determined burn-in length reproduces the paper's reported value: 250 iterations for both Cosmo Hammer and fine-tuned CosmoMC configurations.
- After the burn-in point, X̄_t remains fluctuating within the predicted sigma/sqrt(L) band for all subsequent iterations.
- The self-tuning CosmoMC tracking reproduces the reported ~2000-step proposal-tuning phase before samples become usable.
- The walkers ratio maps correctly to walker count: walkers ratio 50 with 6 parameters yields 50 × 6 = 300 walkers, and L used in sigma/sqrt(L) equals that walker/chain count.
- The output artefacts exist and match the expected schema: burnin_convergence_analysis.png (trajectories with band) and burnin_summary.csv (burn-in lengths table).

## Limitations

- Self-tuning CosmoMC chains must first pass a roughly 2000-step proposal-tuning phase; tracking burn-in on tuning-phase samples is invalid.
- The criterion assumes the ensemble mean behaves like sigma/sqrt(L) only after initialisation bias is gone; strongly correlated walkers (different autocorrelation times between Cosmo Hammer and CosmoMC) still converge to the same safe burn-in here but the comparison relies on the scatter across realisations.
- Scaling caveats of the underlying sampler apply: processes idle when more processes than walkers are available, and scaling shows ~11% overhead per core doubling (alpha = 0.89), which limits how far L can be pushed in the comparison.

## Evidence

- [other] By monitoring the mean sequential step of the walkers against the predicted standard error of the mean, a safe burn-in period of 250 iterations is found for both Cosmo Hammer and fine-tuned CosmoMC, despite their different autocorrelation times.: [evidence span withheld — anchor status: not_found]
- [discussion] Create CosmoHammerSampler with params, chain, file prefix, walkers ratio, burn-in and sample iterations, then call startSampling(): [evidence span withheld — anchor status: not_found]
- [discussion] A walkers Ratio of 50 will launch 50 × 6 = 300 walkers, where six is the number of parameters sampled.: "A walkers Ratio of 50 will launch 50 × 6 = 300 walkers, where six is the number of parameters sampled."
- [readme] CosmoHammer is a framework which embeds emcee, an implementation by Foreman-Mackey et al. (2012) of the Affine Invariant Markov chain Monte Carlo (MCMC) Ensemble sampler: "CosmoHammer is a framework which embeds `emcee <http://arxiv.org/abs/1202.3665>`_ , an implementation by Foreman-Mackey et al. (2012)"
- [other] Repeat the same tracking for the self-tuning CosmoMC Metropolis-Hastings chains, which require a roughly 2000-step proposal-tuning phase before their samples become usable.: [evidence span withheld — anchor status: not_found]
