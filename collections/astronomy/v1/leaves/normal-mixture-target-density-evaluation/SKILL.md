---
name: normal-mixture-target-density-evaluation
description: Reference for testing whether a population-based MCMC method converges
  on a bimodal, distant-mode target where random walk Metropolis fails; triggered
  when RMSE of the posterior mean estimate and acceptance fraction must be quantified
  for d=5 and d=10 mixtures.
when_to_use_negative:
- If the initial population covers just a single modal region — distant modes cannot
  be reached, so the mixture benchmark result is invalid
- When the target is unimodal or low-dimensional where ordinary one-dimensional MCMC
  updates (e.g. WinBUGS-style) suffice and the mixture stress test adds no information
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: demc2
  role: R library providing Differential Evolution Markov Chain adaptive MCMC used
    to sample the normal mixture target
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: R package used to install the demc2 implementation from GitHub
- name: R
  role: Language in which the mixture target density, overdispersed chain initialization,
    and RMSE/acceptance-fraction evaluation are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:7e1edbf9c6e6881d2ca9d8d7130cc3591126e89d163c7be0d9d81b056e21c74e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - TraitEnvMLMWA
  - demc2
  - remotes
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# normal-mixture-target-density-evaluation

## Summary

Benchmark a multi-chain MCMC sampler such as DE-MC on the notoriously difficult bimodal normal mixture target (1/3)N_d(-5,I)+(2/3)N_d(5,I) in R, checking convergence of the chain mean to the true posterior mean 5/3 ≈ 1.667 and comparing RMSE and acceptance fraction across dimensions and gamma settings.

## When to use

Reference for testing whether a population-based MCMC method converges on a bimodal, distant-mode target where random walk Metropolis fails; triggered when RMSE of the posterior mean estimate and acceptance fraction must be quantified for d=5 and d=10 mixtures.

## When NOT to use

- If the initial population covers just a single modal region — distant modes cannot be reached, so the mixture benchmark result is invalid
- When the target is unimodal or low-dimensional where ordinary one-dimensional MCMC updates (e.g. WinBUGS-style) suffice and the mixture stress test adds no information

## Inputs

- Normal mixture target density definition (1/3)N_d(-5,I)+(2/3)N_d(5,I) in R for d=5 and d=10
- Overdispersed initial population of chains covering both mixture modes
- Jump scale settings: default gamma = 2.38/sqrt(2d) and adapted gamma = 1.0 every tenth generation

## Outputs

- RMSE of the posterior mean estimate for d=5 (≈0.023 default, 0.015 adapted) and d=10 (≈0.009)
- Acceptance fraction (≈0.16 for d=5, ≈0.15 for d=10)
- demc2_mixture_benchmark_results.csv with RMSE values and acceptance fractions for both dimensions

## How to apply

Install the sampler from GitHub with remotes::install_github("CajoterBraak/demc2") after install.packages("remotes"). Define the target mixture density (1/3)N_d(-5,I)+(2/3)N_d(5,I) in R for d=5 and d=10, noting the true posterior mean is 5/3 ≈ 1.667. Initialize a population of chains from overdispersed starting states covering both mixture modes, because a population confined to one modal region cannot reach distant modes. Run d=5 with the default jump scale gamma = 2.38/sqrt(2d) and record the RMSE of the posterior mean estimate (target ≈ 0.023; improved variant target 0.015, with acceptance fraction ~0.16). Run d=10 setting gamma = 1.0 every tenth generation, then verify convergence of the chain mean to 1.667, RMSE ≈ 0.009, and an acceptance fraction ≈ 0.15. Write the RMSE values and acceptance fractions for both dimensions to demc2_mixture_benchmark_results.csv.

## Related tools

- **demc2** (R library providing Differential Evolution Markov Chain adaptive MCMC used to sample the normal mixture target) — https://github.com/CajoterBraak/demc2
- **remotes** (R package used to install the demc2 implementation from GitHub)
- **R** (Language in which the mixture target density, overdispersed chain initialization, and RMSE/acceptance-fraction evaluation are implemented)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Chain mean for d=10 with gamma=1.0 every tenth generation converges to the expected value 1.667
- RMSE of the posterior mean estimate is ≈0.009 for d=10 (adapted gamma) and ≈0.023 (default) / 0.015 (adapted) for d=5
- Acceptance fraction falls near 0.15 for d=10 and ~0.16 for d=5
- demc2_mixture_benchmark_results.csv exists and contains RMSE values and acceptance fractions for both d=5 and d=10
- Estimated mean deviates substantially from 5/3 if the initial population was confined to a single modal region, indicating a setup error

## Limitations

- If the initial population covers just a single modal region, distant modes cannot be reached
- Large populations require more computer time to converge than small ones
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence; gamma may need to be decreased if blocks are strongly correlated

## Evidence

- [results] With gamma = 1.0 every tenth generation, DE-MC converged to the expected value 1.667 with a RMSE of 0.009 and an acceptance fraction of 0.15: [evidence span withheld — anchor status: not_found]
