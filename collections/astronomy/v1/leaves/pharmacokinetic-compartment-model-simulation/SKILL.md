---
name: pharmacokinetic-compartment-model-simulation
description: Use when fitting a Bayesian nonlinear mixed-effects PK compartment model
  (e.g. Theophylline) where one-dimensional updates are inadequate and posterior percentiles
  of location and variance parameters are needed.
when_to_use_negative:
- The model lies within the generalized linear model context, where WinBUGS is available
  and adequate — outside GLMs WinBUGS is limited to one-dimensional updates, which
  is precisely when DE-MC block updating is preferred
- 'The initial population of chains covers only a single modal region of a multimodal
  posterior: distant modes cannot be reached by DE-MC'
- Only a greedy optimizer is needed (e.g. pure global optimization without posterior
  sampling); DE-MC is not a greedy algorithm, so DE-variants of simulated annealing
  or tempering may fit better
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: demc2
  role: R library providing the Differential Evolution Markov Chain adaptive MCMC
    sampler used to fit the Theophylline nonlinear mixed-effects model
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: R package used to install demc2 from GitHub via remotes::install_github('CajoterBraak/demc2')
- name: R
  role: Runtime environment in which the Theophylline dataset (nlme) is loaded and
    DE-MC sampling is executed
- name: WinBUGS
  role: Provides the very long reference run against which DE-MC posterior percentiles
    are compared (Table 4 of the paper)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:21d939d1bacbd66eab719636ac0ec9355543bc8f0097a92713d4b540f6221361
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - TraitEnvMLMWA
  - WinBUGS
  - demc2
  - remotes
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pharmacokinetic-compartment-model-simulation

## Summary

Fit and sample a 43-parameter Bayesian nonlinear mixed-effects first-order open-compartment pharmacokinetic model (Theophylline data from Pinheiro & Bates p.444) using Differential Evolution Markov Chain (DE-MC), then validate posterior percentiles against a very long WinBUGS reference run.

## When to use

Use when fitting a Bayesian nonlinear mixed-effects PK compartment model (e.g. Theophylline) where one-dimensional updates are inadequate and posterior percentiles of location and variance parameters are needed.

## When NOT to use

- The model lies within the generalized linear model context, where WinBUGS is available and adequate — outside GLMs WinBUGS is limited to one-dimensional updates, which is precisely when DE-MC block updating is preferred
- The initial population of chains covers only a single modal region of a multimodal posterior: distant modes cannot be reached by DE-MC
- Only a greedy optimizer is needed (e.g. pure global optimization without posterior sampling); DE-MC is not a greedy algorithm, so DE-variants of simulated annealing or tempering may fit better

## Inputs

- Theophylline dataset from the R nlme package (Pinheiro & Bates p.444)
- 43-parameter Bayesian nonlinear mixed-effects first-order open-compartment model specification with priors on l Ke, l Ka, l Cl, log(tau2_e/a/c), and log(sigma2)
- Very long WinBUGS reference run percentiles for validation

## Outputs

- Pooled post-burn-in posterior samples for the 43 parameters
- P2.5, P50, P97.5 percentile table for l Ke, l Ka, l Cl, log(tau2_e/a/c), and log(sigma2) saved as demc_theophylline_percentiles.csv

## How to apply

Install the DE-MC implementation via install.packages('remotes') and remotes::install_github('CajoterBraak/demc2'). Load the Theophylline dataset from the nlme package and specify first-order open-compartment model priors on l Ke, l Ka, l Cl, log(tau2_e/a/c), and log(sigma2). Initialize N = 2d = 86 chains from overdispersed starting states across the 43-dimensional space; at each of 50,000 generations propose a Differential Evolution jump adding gamma = 2.38/sqrt(2d) times the difference vector of two randomly chosen other chains, accepting or rejecting per chain with the Metropolis ratio so jumps adapt to the target variance-covariance matrix. Decrease gamma somewhat if needed to reach an optimal Metropolis acceptance rate, as was required for this nonlinear mixed-effects model. Compute P2.5, P50, and P97.5 percentiles from pooled post-burn-in samples and compare against the very long WinBUGS reference column (Table 4 of the paper), saving as demc_theophylline_percentiles.csv.

## Related tools

- **demc2** (R library providing the Differential Evolution Markov Chain adaptive MCMC sampler used to fit the Theophylline nonlinear mixed-effects model) — https://github.com/CajoterBraak/demc2
- **remotes** (R package used to install demc2 from GitHub via remotes::install_github('CajoterBraak/demc2'))
- **R** (Runtime environment in which the Theophylline dataset (nlme) is loaded and DE-MC sampling is executed)
- **WinBUGS** (Provides the very long reference run against which DE-MC posterior percentiles are compared (Table 4 of the paper))

## Examples

```
install.packages('remotes'); remotes::install_github('CajoterBraak/demc2')
```

## Evaluation signals

- DE-MC P2.5/P50/P97.5 percentiles agree with the very long WinBUGS reference column (Table 4) within Monte Carlo error
- Root mean squared error for location parameters (l Ke, l Ka, l Cl) is up to a factor of 4 lower than WinBUGS, while variance parameters may be up to a factor 2 higher
- Metropolis acceptance rate is near optimal, with gamma = 2.38/sqrt(2d) decreased somewhat if the nonlinear mixed-effects model demands it
- N = 2d = 86 chains initialized from overdispersed starting states across the 43-dimensional parameter space
- Percentile table written to demc_theophylline_percentiles.csv with columns for all five reported parameter groups

## Limitations

- No theoretical analysis of the rate of convergence of DE-MC exists yet; the authors state it is much desired
- If the initial population covers just a single modal region, distant modes cannot be reached
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence
- Large populations require more computer time to converge than small ones
- In the nonlinear mixed-effects model, gamma needed to be decreased somewhat to get an optimal acceptance rate
- The conditional distribution required for Gibbs sampling of gamma is often not available in closed form or not easy to sample from directly

## Evidence

- [results] Compared to WinBUGS on the Theophylline model, DE-MC achieved a root mean squared error up to a factor of 4 lower for the location parameters (l Ke, l Ka, l Cl): [evidence span withheld — anchor status: not_found]
- [methods] Load the Theophylline dataset from the nlme package: [evidence span withheld — anchor status: not_found]
