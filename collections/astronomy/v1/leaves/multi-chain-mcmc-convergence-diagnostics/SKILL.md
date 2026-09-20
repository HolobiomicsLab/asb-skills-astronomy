---
name: multi-chain-mcmc-convergence-diagnostics
description: Use when sampling a real-parameter posterior (e.g. a d=5 or d=10 normal
  mixture with modes at -5 and 5) and you need to verify that chains from dispersed
  starts mix to the same target distribution and recover a known posterior mean.
when_to_use_negative:
- The initial population covers only a single modal region of a multimodal target
  — distant modes cannot be reached, so convergence diagnostics are meaningless
- The problem is a one-dimensional update within a generalized linear model, where
  WinBUGS-style independent chains suffice and the population-based approach adds
  cost
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: demc2
  role: R package implementing DE-MC sampling with multiple parallel chains for convergence
    checking
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: Used to install the demc2 sampler from GitHub
- name: R
  role: Environment in which the target density is defined and DE-MC is run
- name: WinBUGS
  role: 'Contrasted approach: runs chains independently for convergence checking and
    is limited to one-dimensional updates outside GLMs'
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:d319455127bc516fd6b5f422969bc4c6eba72610667a725f2ae66731f2d2558f
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
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# multi-chain-mcmc-convergence-diagnostics

## Summary

Convergence assessment for MCMC by running multiple Markov chains in parallel from overdispersed starting states and letting them learn from each other, as implemented in the DE-MC sampler, rather than running chains independently as in WinBUGS-style checks.

## When to use

Use when sampling a real-parameter posterior (e.g. a d=5 or d=10 normal mixture with modes at -5 and 5) and you need to verify that chains from dispersed starts mix to the same target distribution and recover a known posterior mean.

## When NOT to use

- The initial population covers only a single modal region of a multimodal target — distant modes cannot be reached, so convergence diagnostics are meaningless
- The problem is a one-dimensional update within a generalized linear model, where WinBUGS-style independent chains suffice and the population-based approach adds cost

## Inputs

- Target posterior density defined in R (e.g. two-component normal mixture (1/3)N_d(-5,I)+(2/3)N_d(5,I))
- Overdispersed initial states for a population of chains covering both mixture modes
- Number of parameters d (e.g. 5 or 10) for setting gamma = 2.38/sqrt(2d)

## Outputs

- Per-chain posterior samples from converged runs
- Posterior mean estimates with RMSE values (e.g. 0.023, 0.015, 0.009) and acceptance fractions (≈0.15)
- Results table such as demc2_mixture_benchmark_results.csv

## How to apply

Initialize a population of chains from overdispersed starting states that cover all modal regions — for the mixture target (1/3)N_d(-5,I)+(2/3)N_d(5,I) both modes must be represented, since a population confined to one modal region cannot reach distant modes. Run DE-MC with the default jump scale gamma = 2.38/sqrt(2d), decreasing gamma if blocks are strongly correlated or, for d=10, setting gamma = 1.0 every tenth generation. Judge convergence by checking that all chains from overdispersed starts mix to the same target distribution, that the chain mean converges to the expected value (1.667 for the mixture), and by RMSE of the posterior mean estimate (≈0.023 at d=5 default gamma, 0.015 with adaptation, 0.009 at d=10) and acceptance fraction (≈0.15–0.16).

## Related tools

- **demc2** (R package implementing DE-MC sampling with multiple parallel chains for convergence checking) — https://github.com/CajoterBraak/demc2
- **remotes** (Used to install the demc2 sampler from GitHub)
- **R** (Environment in which the target density is defined and DE-MC is run)
- **WinBUGS** (Contrasted approach: runs chains independently for convergence checking and is limited to one-dimensional updates outside GLMs)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Chain mean converges to the true posterior mean 5/3 ≈ 1.667 for the normal mixture target
- RMSE of the posterior mean falls in the reported range: ≈0.023 at d=5 with default gamma, 0.015 with gamma=1.0 adaptation, and ≈0.009 at d=10
- Acceptance fraction is ≈0.15–0.16 as reported for the mixture benchmark
- All chains initialized from overdispersed states mix to the same target distribution
- Acceptance rate is near optimal; decrease gamma if blocks are strongly correlated or acceptance is suboptimal

## Limitations

- If the initial population covers just a single modal region, distant modes cannot be reached
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence
- Large populations require more computer time to converge than small ones
- No theoretical analysis of the rate of convergence of DE-MC exists yet
- Gamma may need to be decreased if blocks are strongly correlated or to reach an optimal acceptance rate (as in the nonlinear mixed-effects model)

## Evidence
