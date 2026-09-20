---
name: bayesian-nonlinear-mixed-effects-modeling
description: Use when a Bayesian nonlinear mixed-effects model has ~40+ parameters
  (e.g. l Ke, l Ka, l Cl, log(tau2_e/a/c), log(sigma2)) where one-dimensional Gibbs/WinBUGS
  updates are limited and block MCMC with adaptive jump scale and orientation is needed.
when_to_use_negative:
- When the target distribution's modes are far apart and the initial population covers
  only a single modal region — distant modes cannot be reached
- When a greedier optimizer is needed for fast convergence; DE-MC is not greedy and
  is a disadvantage in terms of speed of convergence
- When running a very long reference run (e.g. WinBUGS) is acceptable and a validated
  single-chain sampler is preferred for convergence checking in the Gelman et al.
  / WinBUGS style
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: demc2 (R package)
  role: Differential Evolution Markov Chain sampler used to generate posterior samples
    of the 43-parameter Theophylline model
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: R package installer used to install demc2 from GitHub
- name: R
  role: Runtime environment for model specification, MCMC execution, and percentile
    computation
- name: WinBUGS
  role: External reference sampler producing very long posterior runs used to validate
    DE-MC percentiles within Monte Carlo error
- name: nlme
  role: Source of the Theophylline dataset used to fit the pharmacokinetic nonlinear
    mixed-effects model
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:73f329c2a021f954fff665b817b359e9e4144bdff47a10295430984e063c3389
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - TraitEnvMLMWA
  - WinBUGS
  - demc2 (R package)
  - nlme
  - remotes
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# bayesian-nonlinear-mixed-effects-modeling

## Summary

Fit a high-dimensional Bayesian nonlinear mixed-effects model (e.g. the 43-parameter Theophylline first-order open-compartment pharmacokinetic model) using DE-MC, whose N parallel chains supply scale- and orientation-adaptive Metropolis jumps without closed-form conditional distributions.

## When to use

Use when a Bayesian nonlinear mixed-effects model has ~40+ parameters (e.g. l Ke, l Ka, l Cl, log(tau2_e/a/c), log(sigma2)) where one-dimensional Gibbs/WinBUGS updates are limited and block MCMC with adaptive jump scale and orientation is needed.

## When NOT to use

- When the target distribution's modes are far apart and the initial population covers only a single modal region — distant modes cannot be reached
- When a greedier optimizer is needed for fast convergence; DE-MC is not greedy and is a disadvantage in terms of speed of convergence
- When running a very long reference run (e.g. WinBUGS) is acceptable and a validated single-chain sampler is preferred for convergence checking in the Gelman et al. / WinBUGS style

## Inputs

- Nonlinear mixed-effects model specification with priors (e.g. on l Ke, l Ka, l Cl, log(tau2_e/a/c), log(sigma2))
- Pharmacokinetic time-course data such as the Theophylline dataset from the nlme package (Pinheiro & Bates p.444)
- Overdispersed starting states for N = 2d chains across the d-dimensional parameter space
- Optional external reference posterior percentiles (e.g. very long WinBUGS run) for validation

## Outputs

- Pooled post-burn-in MCMC samples of all 43 parameters
- P2.5, P50, P97.5 percentiles for each parameter (saved e.g. as demc_theophylline_percentiles.csv)
- Root mean squared error of percentiles versus a reference run (WinBUGS), for comparison

## How to apply

Install the sampler via install.packages('remotes') then remotes::install_github('CajoterBraak/demc2'). Specify the nonlinear mixed-effects model with priors on l Ke, l Ka, l Cl, log(tau2_e/a/c), and log(sigma2) (Theophylline data from the nlme package, Pinheiro & Bates p.444). Initialize N = 2d = 86 chains from overdispersed states across the 43-dimensional space, and at each of 50,000 generations propose a Differential Evolution jump: current chain vector plus gamma times the difference of two randomly chosen other chains, with gamma = 2.38/sqrt(2d) by default. Accept or reject each block proposal with the Metropolis ratio; the population thereby lets jump scale and orientation adapt automatically to the variance-covariance matrix of the target. If acceptance is suboptimal for a nonlinear mixed-effects model, decrease gamma somewhat from the default. Pool post-burn-in samples and report P2.5, P50, P97.5 percentiles of the location and variance parameters.

## Related tools

- **demc2 (R package)** (Differential Evolution Markov Chain sampler used to generate posterior samples of the 43-parameter Theophylline model) — https://github.com/CajoterBraak/demc2
- **remotes** (R package installer used to install demc2 from GitHub)
- **R** (Runtime environment for model specification, MCMC execution, and percentile computation)
- **WinBUGS** (External reference sampler producing very long posterior runs used to validate DE-MC percentiles within Monte Carlo error)
- **nlme** (Source of the Theophylline dataset used to fit the pharmacokinetic nonlinear mixed-effects model)

## Examples

```
install.packages('remotes'); remotes::install_github('CajoterBraak/demc2')
```

## Evaluation signals

- DE-MC P2.5/P50/P97.5 percentiles agree with a very long WinBUGS reference column (Table 4 of the paper) within Monte Carlo error
- Root mean squared error of DE-MC percentiles versus WinBUGS is up to a factor of 4 lower for location parameters (l Ke, l Ka, l Cl) and up to a factor 2 higher for variance parameters
- Metropolis acceptance rate is tuned to its optimal value (decrease gamma somewhat from 2.38/sqrt(2d) if needed for the nonlinear mixed-effects model)
- Pooled percentiles for l Ke, l Ka, l Cl, log(tau2_e/a/c), and log(sigma2) are computed from post-burn-in samples of all N = 2d = 86 chains
- Pooled percentiles table is saved as demc_theophylline_percentiles.csv

## Limitations

- Gamma may need to be decreased if blocks are strongly correlated, beyond the 2.38/sqrt(2d) default
- Large chain populations (N) require more computer time to converge than small ones
- No theoretical analysis of the rate of convergence of DE-MC exists yet
- If the initial population covers just a single modal region, distant modes cannot be reached
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence
- WinBUGS (the reference comparator) is limited to one-dimensional updates outside the generalized linear model context

## Evidence

- [discussion] setup_gamma_and_population: "The suggested default value of γ = 2 . 38 / √ (2d) performed well in bock updating (Table 4)"
- [discussion] adaptation_rationale: "The scale and orientation of the jumps in DE-MC (2) automatically adapt themselves to the variance-covariance matrix of the target distribution"
- [discussion] gamma_tuning_for_mixed_models: "In the nonlinear mixed-effects model, γ needed to be decreased somewhat to get an optimal accep- tance rate."
- [readme] install_command: "remotes::install_github("CajoterBraak/demc2")"
- [intro] winbugs_reference_role: "instead of running the chains independently as a way to check convergence. (Gel- man et al., 2004) and as carried out in WinBUGS (Lunn et al., 2000)"
