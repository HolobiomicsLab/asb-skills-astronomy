---
name: cross-sampler-convergence-comparison-against-bugs-reference
description: Use when a newly implemented MCMC sampler (e.g. DE-MC with N=2d chains,
  gamma=2.38/sqrt(2d)) must be validated by reproducing posterior percentiles of a
  high-dimensional Bayesian model against an independent long-run reference sampler
  like WinBUGS.
when_to_use_negative:
- The model lives in a generalized linear model context where WinBUGS itself performs
  well with standard one-dimensional updates, making a cross-sampler benchmark redundant
- The initial chain population covers only a single modal region of a multimodal posterior
  — distant modes cannot be reached, so percentile agreement would be misleading
- No trustworthy long-run reference exists; comparing two unvalidated samplers does
  not establish convergence
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3489
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Environment for loading the Theophylline dataset (nlme package) and computing
    pooled posterior percentiles from DE-MC samples
- name: remotes
  role: Used to install the DE-MC sampler implementation from GitHub (remotes::install_github('CajoterBraak/demc2'))
- name: WinBUGS
  role: Provides the very long independent reference run against which DE-MC posterior
    percentiles are compared
- name: demc2
  role: R implementation of Differential Evolution Markov Chain used to generate the
    test-sampler posterior samples
  repo: CajoterBraak/demc2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:8c68a6289caf0c73fcce13b04965c0a53343d4c2c1ee1a1194a3b5ef14ce51c4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3489
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# cross-sampler-convergence-comparison-against-bugs-reference

## Summary

Validate a new MCMC sampler (e.g. DE-MC on a 43-parameter Theophylline nonlinear mixed-effects model) by comparing its posterior percentiles (P2.5/P50/P97.5) against a very long reference run from an independent sampler such as WinBUGS, within Monte Carlo error.

## When to use

Use when a newly implemented MCMC sampler (e.g. DE-MC with N=2d chains, gamma=2.38/sqrt(2d)) must be validated by reproducing posterior percentiles of a high-dimensional Bayesian model against an independent long-run reference sampler like WinBUGS.

## When NOT to use

- The model lives in a generalized linear model context where WinBUGS itself performs well with standard one-dimensional updates, making a cross-sampler benchmark redundant
- The initial chain population covers only a single modal region of a multimodal posterior — distant modes cannot be reached, so percentile agreement would be misleading
- No trustworthy long-run reference exists; comparing two unvalidated samplers does not establish convergence

## Inputs

- Posterior samples from the sampler under test (pooled post-burn-in draws from N=2d chains, e.g. 86 chains x 50,000 generations)
- Very long reference run posterior samples from WinBUGS for the same model and data
- Model specification with priors (e.g. Theophylline 43-parameter first-order open-compartment nonlinear mixed-effects model; Theoph dataset from the nlme package, Pinheiro & Bates p.444)

## Outputs

- Table of P2.5/P50/P97.5 percentiles per parameter with test-sampler and reference columns (e.g. demc_theophylline_percentiles.csv)
- Per-parameter-type root mean squared error of percentile deviations (location vs variance parameters)

## How to apply

Fit the same model twice: once with the sampler under test (e.g. DE-MC, initialized from N=2d=86 overdispersed chains, run 50,000 generations with proposals formed by gamma times the difference vector of two randomly chosen other chains, tuned to an optimal Metropolis acceptance rate — decrease gamma if blocks are strongly correlated), and once with an independent reference sampler (very long WinBUGS run). Compute P2.5, P50, and P97.5 percentiles of the parameters of interest (here l Ke, l Ka, l Cl, log(tau2_e/a/c), and log(sigma2)) from pooled post-burn-in samples of the test sampler. Compare test-sampler percentiles against the reference column within Monte Carlo error, optionally summarizing deviations as root mean squared error per parameter type (the paper found RMSE up to 4x lower for location parameters l Ke, l Ka, l Cl and up to 2x higher for variance parameters vs WinBUGS). Save the comparison table (e.g. demc_theophylline_percentiles.csv).

## Related tools

- **R** (Environment for loading the Theophylline dataset (nlme package) and computing pooled posterior percentiles from DE-MC samples)
- **remotes** (Used to install the DE-MC sampler implementation from GitHub (remotes::install_github('CajoterBraak/demc2')))
- **WinBUGS** (Provides the very long independent reference run against which DE-MC posterior percentiles are compared)
- **demc2** (R implementation of Differential Evolution Markov Chain used to generate the test-sampler posterior samples) — CajoterBraak/demc2

## Examples

```
install.packages('remotes'); remotes::install_github('CajoterBraak/demc2')
```

## Evaluation signals

- DE-MC P2.5/P50/P97.5 percentiles of l Ke, l Ka, l Cl, log(tau2_e/a/c), and log(sigma2) agree with the very long WinBUGS reference column (Table 4 of the paper) within Monte Carlo error
- Sampler tuned to an optimal Metropolis acceptance rate (decrease gamma = 2.38/sqrt(2d) somewhat if needed, as required for this nonlinear mixed-effects model)
- Root mean squared error of percentile deviations is of the order reported (RMSE up to 4x lower for location parameters, up to 2x higher for variance parameters vs WinBUGS)
- Comparison table saved with both sampler columns present (e.g. demc_theophylline_percentiles.csv)

## Limitations

- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence; large populations require more computer time to converge than small ones
- If the initial population covers just a single modal region, distant modes cannot be reached, so cross-sampler percentile agreement may mask non-convergence
- gamma may need to be decreased if blocks are strongly correlated; in the nonlinear mixed-effects model gamma needed to be decreased somewhat to get an optimal acceptance rate
- No theoretical analysis of the rate of convergence of DE-MC exists yet, so validation is empirical rather than guaranteed

## Evidence

- [results] task card finding: [evidence span withheld — anchor status: not_found]
- [methods] task card validation: [evidence span withheld — anchor status: not_found]
