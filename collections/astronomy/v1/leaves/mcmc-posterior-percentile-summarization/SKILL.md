---
name: mcmc-posterior-percentile-summarization
description: Use when posterior summaries of a Bayesian hierarchical/nonlinear mixed-effects
  model must be reported as percentiles and checked against a reference sampler's
  output for agreement within Monte Carlo error.
when_to_use_negative:
- 'WinBUGS itself is the appropriate tool: outside the generalized linear model context
  WinBUGS is limited to one-dimensional updates, so DE-MC block updating is preferable
  for strongly correlated parameter blocks'
- The target posterior is multimodal with distant modes and the initial population
  covers only a single modal region, since distant modes cannot then be reached
- A theoretical convergence-rate guarantee is required; no such analysis of DE-MC
  exists yet
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: demc2
  role: R implementation of Differential Evolution Markov Chain used to generate the
    pooled posterior samples
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: Used to install the demc2 package from GitHub
- name: R
  role: Environment for running DE-MC, loading the nlme Theophylline data, and computing
    percentiles
- name: WinBUGS
  role: Provides the very long reference run whose posterior percentiles validate
    the DE-MC summaries
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:57c98e20a90fde0a0a06961697262b003cd27e95841316e1af2dbbe04aaf0917
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
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# mcmc-posterior-percentile-summarization

## Summary

Summarize a high-dimensional Bayesian posterior by computing P2.5, P50, and P97.5 percentiles of key parameters (e.g. location and variance parameters of a nonlinear mixed-effects model) from pooled multi-chain MCMC samples, and validate the sampler by comparing these percentiles against an independent long-run reference (e.g. a very long WinBUGS run).

## When to use

Use when posterior summaries of a Bayesian hierarchical/nonlinear mixed-effects model must be reported as percentiles and checked against a reference sampler's output for agreement within Monte Carlo error.

## When NOT to use

- WinBUGS itself is the appropriate tool: outside the generalized linear model context WinBUGS is limited to one-dimensional updates, so DE-MC block updating is preferable for strongly correlated parameter blocks
- The target posterior is multimodal with distant modes and the initial population covers only a single modal region, since distant modes cannot then be reached
- A theoretical convergence-rate guarantee is required; no such analysis of DE-MC exists yet

## Inputs

- Pooled post-burn-in MCMC samples from N = 2d parallel chains (e.g. 86 chains over 43 parameters)
- Bayesian nonlinear mixed-effects model specification with priors on l Ke, l Ka, l Cl, log(tau2_e/a/c), and log(sigma2)
- Theophylline dataset from the nlme package (Pinheiro & Bates p.444)
- Very long WinBUGS reference run percentiles for validation

## Outputs

- Table of P2.5, P50, and P97.5 percentiles per parameter (e.g. demc_theophylline_percentiles.csv)
- Comparison of DE-MC percentiles against the very long WinBUGS reference column

## How to apply

Fit the model with DE-MC (gamma = 2.38/sqrt(2d), N = 2d chains, e.g. 86 chains for the 43-parameter Theophylline model) running for many generations (e.g. 50,000), decreasing gamma somewhat from the default if needed to reach an optimal Metropolis acceptance rate. Pool post-burn-in samples across chains, exploiting that the jump scale and orientation automatically adapt to the variance-covariance matrix of the target distribution. Compute the P2.5, P50, and P97.5 percentiles for the parameters of interest — in the Theophylline case study, l Ke, l Ka, l Cl, log(tau2_e/a/c), and log(sigma2) — and save the result as a comparison table (e.g. demc_theophylline_percentiles.csv). Validate by checking that the DE-MC percentiles agree with a very long WinBUGS reference run within Monte Carlo error, as in Table 4 of the paper, where DE-MC achieved RMSE up to a factor of 4 lower than WinBUGS for the location parameters and up to a factor 2 higher for the variance parameters.

## Related tools

- **demc2** (R implementation of Differential Evolution Markov Chain used to generate the pooled posterior samples) — https://github.com/CajoterBraak/demc2
- **remotes** (Used to install the demc2 package from GitHub)
- **R** (Environment for running DE-MC, loading the nlme Theophylline data, and computing percentiles)
- **WinBUGS** (Provides the very long reference run whose posterior percentiles validate the DE-MC summaries)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- DE-MC P2.5/P50/P97.5 percentiles agree with the very long WinBUGS reference percentiles within Monte Carlo error (Table 4 criterion)
- Root mean squared error of DE-MC percentile estimates versus reference is up to a factor 4 lower than WinBUGS for location parameters (l Ke, l Ka, l Cl) and no more than a factor 2 higher for variance parameters
- Metropolis acceptance rate is tuned to its optimum, decreasing gamma from 2.38/sqrt(2d) if needed for the nonlinear mixed-effects model
- Percentile table (e.g. demc_theophylline_percentiles.csv) contains rows for l Ke, l Ka, l Cl, log(tau2_e/a/c), and log(sigma2) with all three percentiles populated

## Limitations

- If blocks are strongly correlated, gamma may need to be decreased below the default 2.38/sqrt(2d); in the nonlinear mixed-effects model gamma needed to be decreased somewhat to get an optimal acceptance rate
- If the initial population covers just a single modal region, distant modes cannot be reached
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence, and large populations require more computer time to converge than small ones
- No theoretical analysis of the rate of convergence of DE-MC exists yet

## Evidence

- [methods] Percentile computation from pooled samples: [evidence span withheld — anchor status: not_found]
- [results] Validation against WinBUGS reference: [evidence span withheld — anchor status: not_found]
