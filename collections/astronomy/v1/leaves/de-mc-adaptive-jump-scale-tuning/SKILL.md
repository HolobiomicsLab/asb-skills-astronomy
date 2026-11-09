---
name: de-mc-adaptive-jump-scale-tuning
description: Use when evaluating or tuning the jump scale gamma of Differential Evolution
  Markov Chain (DE-MC) on a multimodal target such as a two-component Normal mixture,
  where random walk Metropolis fails and RMSE of the posterior mean and acceptance
  fraction are the quality metrics.
when_to_use_negative:
- The target is unimodal and low-dimensional where random walk Metropolis or standard
  Gibbs sampling suffices — DE-MC's population machinery and gamma tuning add unnecessary
  overhead
- 'The initial population can only cover a single modal region: as the paper warns,
  ''if the initial population covers just a single modal region, there is no chance
  that other modes that are far away can be reached'''
- Blocks of parameters are strongly correlated and gamma has not been decreased —
  the paper notes gamma may need to be decreased in that case, so untuned default
  runs would be misleading
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: demc2
  role: R implementation of Differential Evolution Markov Chain used to sample the
    bimodal normal mixture target and measure RMSE and acceptance fraction
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: R package used to install demc2 from GitHub via remotes::install_github('CajoterBraak/demc2')
- name: R
  role: Runtime environment in which the mixture target density is defined, DE-MC
    is run, and results are written to CSV
- name: TraitEnvMLMWA
  role: Referenced in the demc2 README installation instructions as the released-version
    package install pattern from GitHub
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:d5683aecef704d1441fa457b0cd2d7f1452482e1e40df674e5ad52c13a364988
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - TraitEnvMLMWA
  - demc2
  - remotes
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# de-mc-adaptive-jump-scale-tuning

## Summary

Benchmarks the DE-MC sampler (demc2 R package) on the notoriously difficult bimodal normal mixture target (1/3)N_d(-5,I)+(2/3)N_d(5,I), comparing the default jump scale gamma = 2.38/sqrt(2d) against an adapted gamma = 1.0 set every tenth generation, and reports RMSE of the posterior mean and acceptance fraction.

## When to use

Use when evaluating or tuning the jump scale gamma of Differential Evolution Markov Chain (DE-MC) on a multimodal target such as a two-component Normal mixture, where random walk Metropolis fails and RMSE of the posterior mean and acceptance fraction are the quality metrics.

## When NOT to use

- The target is unimodal and low-dimensional where random walk Metropolis or standard Gibbs sampling suffices — DE-MC's population machinery and gamma tuning add unnecessary overhead
- The initial population can only cover a single modal region: as the paper warns, 'if the initial population covers just a single modal region, there is no chance that other modes that are far away can be reached'
- Blocks of parameters are strongly correlated and gamma has not been decreased — the paper notes gamma may need to be decreased in that case, so untuned default runs would be misleading

## Inputs

- Two-component Normal mixture target density (1/3)N_d(-5,I)+(2/3)N_d(5,I) defined in R for d=5 and d=10
- Population of DE-MC chains initialized from overdispersed starting states covering both mixture modes
- Jump scale settings: default gamma = 2.38/sqrt(2d), and gamma = 1.0 applied every tenth generation

## Outputs

- RMSE of the posterior mean estimate relative to the true mean 1.667 (≈ 0.023 for d=5 default gamma; 0.015 for d=5 adapted; ≈ 0.009 for d=10)
- Acceptance fraction (≈ 0.15 for d=10 with gamma = 1.0 every tenth generation)
- demc2_mixture_benchmark_results.csv containing RMSE values and acceptance fractions for both dimensions

## How to apply

Install demc2 from GitHub via remotes::install_github('CajoterBraak/demc2'), then define the mixture density (1/3)N_d(-5,I)+(2/3)N_d(5,I) in R for d=5 and d=10, whose true posterior mean is 5/3 ≈ 1.667. Initialize a population of chains from overdispersed starting states covering both mixture modes, since a population confined to one modal region cannot reach distant modes. For d=5 run with the default gamma = 2.38/sqrt(2d) and record the RMSE of the posterior mean estimate (≈ 0.023); the adaptation reduces it to 0.015. For d=10 set gamma = 1.0 every tenth generation and verify convergence of the chain mean to 1.667 with RMSE ≈ 0.009 and acceptance fraction ≈ 0.15. Write the RMSE values and acceptance fractions for both dimensions to demc2_mixture_benchmark_results.csv.

## Related tools

- **demc2** (R implementation of Differential Evolution Markov Chain used to sample the bimodal normal mixture target and measure RMSE and acceptance fraction) — https://github.com/CajoterBraak/demc2
- **remotes** (R package used to install demc2 from GitHub via remotes::install_github('CajoterBraak/demc2'))
- **R** (Runtime environment in which the mixture target density is defined, DE-MC is run, and results are written to CSV)
- **TraitEnvMLMWA** (Referenced in the demc2 README installation instructions as the released-version package install pattern from GitHub)

## Examples

```
remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- For d=10 with gamma = 1.0 every tenth generation, the chain mean converges to the true posterior mean 1.667 (= 5/3) with RMSE ≈ 0.009
- For d=5 with the default gamma = 2.38/sqrt(2d), the RMSE of the posterior mean is ≈ 0.023, dropping to 0.015 with the adapted gamma
- Acceptance fraction for d=10 with gamma = 1.0 adaptation is ≈ 0.15
- The output file demc2_mixture_benchmark_results.csv contains RMSE values and acceptance fractions for both d=5 and d=10
- Chain diagnostics confirm chains visited both modes of the mixture (means near -5 and 5 regions), i.e. the overdispersed initialization covered both modal regions

## Limitations

- If the initial population covers just a single modal region, there is no chance that other modes that are far away can be reached
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence; finding greedier variants that retain robustness and simplicity is an open challenge
- Large populations require more computer time to converge than small ones
- If blocks are strongly correlated, gamma may need to be decreased; in the nonlinear mixed-effects model gamma needed to be decreased somewhat to get an optimal acceptance rate
- No theoretical analysis of the rate of convergence of DE-MC exists yet; the authors state it is much desired

## Evidence
