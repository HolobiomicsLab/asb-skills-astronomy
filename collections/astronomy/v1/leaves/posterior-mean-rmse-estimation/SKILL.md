---
name: posterior-mean-rmse-estimation
description: Use when validating or tuning a DE-MC / differential-evolution MCMC implementation
  on a known target where the true posterior mean is computable, especially the bimodal
  normal mixture that defeats random walk Metropolis.
when_to_use_negative:
- The target has only a single modal region and the population starts there — the
  benchmark's failure mode (unreachable distant modes) does not apply, and simpler
  unimodal diagnostics suffice
- The true posterior mean is not analytically known, so RMSE against 1.667 cannot
  be computed as a correctness check
- The task is one-dimensional updating in the generalized linear model context, where
  WinBUGS-style per-dimension Gibbs/Metropolis updates are the established choice
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: demc2
  role: R package implementing Differential Evolution Markov Chain used to run the
    population sampler on the mixture target
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: R package used to install demc2 from GitHub via remotes::install_github
- name: R
  role: Runtime environment in which the mixture target is defined, DE-MC is run,
    and RMSE/acceptance-fraction results are written to CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:0fec6836967baac7492c817277962ea64fb3224f56ea8eae64eb6b66ad1ed592
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - TraitEnvMLMWA
  - demc2
  - remotes
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# posterior-mean-rmse-estimation

## Summary

Benchmark a population MCMC sampler (DE-MC) on the two-component Normal mixture target (1/3)N_d(-5,I)+(2/3)N_d(5,I) by measuring RMSE of the posterior mean estimate against the known expected value 1.667, together with the acceptance fraction, for d = 5 and d = 10.

## When to use

Use when validating or tuning a DE-MC / differential-evolution MCMC implementation on a known target where the true posterior mean is computable, especially the bimodal normal mixture that defeats random walk Metropolis.

## When NOT to use

- The target has only a single modal region and the population starts there — the benchmark's failure mode (unreachable distant modes) does not apply, and simpler unimodal diagnostics suffice
- The true posterior mean is not analytically known, so RMSE against 1.667 cannot be computed as a correctness check
- The task is one-dimensional updating in the generalized linear model context, where WinBUGS-style per-dimension Gibbs/Metropolis updates are the established choice

## Inputs

- R target-density function for (1/3)N_d(-5,I)+(2/3)N_d(5,I) at d = 5 and d = 10 (true mean 5/3 ≈ 1.667)
- Overdispersed initial population of MCMC chains covering both mixture modes
- Jump-scale settings: default gamma = 2.38/sqrt(2d) and the adapted gamma = 1.0 every tenth generation

## Outputs

- RMSE of the posterior mean estimate for d = 5 (target ≈ 0.023 default; 0.015 adapted) and d = 10 (target ≈ 0.009)
- Acceptance fraction per configuration (d = 5 ≈ 0.16; d = 10 ≈ 0.15)
- demc2_mixture_benchmark_results.csv with RMSE values and acceptance fractions for both dimensions

## How to apply

Define the mixture density (1/3)N_d(-5,I)+(2/3)N_d(5,I) in R for d = 5 and d = 10, whose true posterior mean is 5/3 ≈ 1.667. Initialize a population of chains from overdispersed starting states that cover both mixture modes — a population confined to one modal region cannot reach distant modes. Run DE-MC (installed from CajoterBraak/demc2) for d = 5 with the default jump scale gamma = 2.38/sqrt(2d), where the expected RMSE is ~0.023 and an adaptation to gamma = 1.0 every tenth generation lowers it to ~0.015. For d = 10 apply gamma = 1.0 every tenth generation and verify convergence of the chain mean to 1.667, RMSE ≈ 0.009, and an acceptance fraction ≈ 0.15. Record the RMSE values and acceptance fractions for both dimensions to a results file (e.g. demc2_mixture_benchmark_results.csv).

## Related tools

- **demc2** (R package implementing Differential Evolution Markov Chain used to run the population sampler on the mixture target) — https://github.com/CajoterBraak/demc2
- **remotes** (R package used to install demc2 from GitHub via remotes::install_github)
- **R** (Runtime environment in which the mixture target is defined, DE-MC is run, and RMSE/acceptance-fraction results are written to CSV)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Chain mean converges to the analytic expected value 1.667 (= 5/3) for the mixture target
- d = 10 run with gamma = 1.0 every tenth generation yields RMSE ≈ 0.009 and acceptance fraction ≈ 0.15
- d = 5 run with default gamma = 2.38/sqrt(2d) yields RMSE ≈ 0.023; the adapted variant lowers it to ≈ 0.015
- Initial population demonstrably covers both mixture modes (chain means do not lock into one modal region)
- demc2_mixture_benchmark_results.csv contains RMSE and acceptance fraction entries for both d = 5 and d = 10

## Limitations

- If the initial population covers just a single modal region, there is no chance that other modes that are far away can be reached
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence; finding greedier variants that retain robustness and simplicity remains an open challenge
- Large populations require more computer time to converge than small ones
- No theoretical analysis of the rate of convergence of DE-MC exists yet
- Gamma may need to be decreased (e.g. gamma = 1.0 every tenth generation) when blocks are strongly correlated or an optimal acceptance rate is needed

## Evidence

- [results] RMSE vs expected posterior mean on mixture target: [evidence span withheld — anchor status: not_found]
- [readme] installation path for demc2: [evidence span withheld — anchor status: not_found]
