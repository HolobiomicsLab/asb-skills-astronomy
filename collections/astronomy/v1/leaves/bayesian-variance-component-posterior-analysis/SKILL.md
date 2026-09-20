---
name: bayesian-variance-component-posterior-analysis
description: Use when a Bayesian one-way random-effects (variance-component) model
  needs posterior percentile tabulation and cross-validation of an MCMC sampler against
  known analytic values or a WinBUGS reference.
when_to_use_negative:
- Outside the generalized linear model context WinBUGS is limited to one-dimensional
  updates, so WinBUGS alone is a poor reference for block-updated high-dimensional
  posteriors — use DE-MC instead
- If the initial population of chains covers just a single modal region, distant modes
  cannot be reached, so the skill is wrong for strongly multimodal targets unless
  the overdispersed initialization spans them
- For strongly correlated blocks where the default gamma = 2.38/sqrt(2d) acceptance
  rate is poor, do not apply the default jump scale unchanged; gamma may need to be
  decreased
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Runtime environment for installing demc2, running the DE-MC chains, and computing/tabulating
    posterior percentiles
- name: remotes
  role: Used to install the demc2 package from the GitHub repository CajoterBraak/demc2
    via remotes::install_github
  repo: https://github.com/CajoterBraak/demc2
- name: TraitEnvMLMWA
  role: Released R package whose GitHub install path is reused to install the demc2
    Differential Evolution Markov Chain library
  repo: https://github.com/CajoterBraak/demc2
- name: WinBUGS
  role: External MCMC reference implementation (version 1.4) whose posterior percentiles
    form the comparison column for validating DE-MC results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:ceba641423f8004cc9c9532c4cf534fa792c157760a56dff3bc873bd4994fca0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - TraitEnvMLMWA
  - WinBUGS
  - remotes
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bayesian-variance-component-posterior-analysis

## Summary

Reproduce and validate posterior percentiles of log(xi) and phi in the Liu & Hodges one-way random-effects (peak-discharge) model using DE-MC overdispersed chain populations, compared against analytically known true percentiles and a WinBUGS 1.4 reference column.

## When to use

Use when a Bayesian one-way random-effects (variance-component) model needs posterior percentile tabulation and cross-validation of an MCMC sampler against known analytic values or a WinBUGS reference.

## When NOT to use

- Outside the generalized linear model context WinBUGS is limited to one-dimensional updates, so WinBUGS alone is a poor reference for block-updated high-dimensional posteriors — use DE-MC instead
- If the initial population of chains covers just a single modal region, distant modes cannot be reached, so the skill is wrong for strongly multimodal targets unless the overdispersed initialization spans them
- For strongly correlated blocks where the default gamma = 2.38/sqrt(2d) acceptance rate is poor, do not apply the default jump scale unchanged; gamma may need to be decreased

## Inputs

- Liu & Hodges peak-discharge one-way random-effects model specification (I = 6 groups, J = 4 observations per group)
- Prior hyperparameters: alpha = 1, beta = 10, a = 1.85, b = 0.1
- Analytically known true percentiles of log(xi) and phi
- WinBUGS 1.4 reference posterior percentiles

## Outputs

- percentiles_comparison.csv with DE-MC posterior percentiles of log(xi) and phi for N = 14, 21, 70 alongside true values and WinBUGS 1.4 reference
- Posterior draws of log(xi) and phi from each DE-MC run

## How to apply

Set up the Liu & Hodges peak-discharge one-way random-effects model with I = 6 groups and J = 4 observations per group under priors alpha = 1, beta = 10, a = 1.85, b = 0.1. Install the demc2 R package via remotes::install_github from CajoterBraak/demc2, initialize an overdispersed population of chains of size N (run N = 14, 21, and 70), and apply the DE-MC jump scale gamma = 2.38/sqrt(2d), using gamma = 1 every 10th generation. Extract posterior draws of log(xi) and phi from each run and compute their percentiles. Tabulate the DE-MC percentiles alongside the analytically known true values and the WinBUGS 1.4 reference column; the N = 14 result should match the true percentiles within 0.02, while the 0.1 discrepancy at the 97.5% point of log(xi) for N = 21 and 70 should vanish with a tenfold longer burn-in. Save the comparison table as percentiles_comparison.csv.

## Related tools

- **R** (Runtime environment for installing demc2, running the DE-MC chains, and computing/tabulating posterior percentiles)
- **remotes** (Used to install the demc2 package from the GitHub repository CajoterBraak/demc2 via remotes::install_github) — https://github.com/CajoterBraak/demc2
- **TraitEnvMLMWA** (Released R package whose GitHub install path is reused to install the demc2 Differential Evolution Markov Chain library) — https://github.com/CajoterBraak/demc2
- **WinBUGS** (External MCMC reference implementation (version 1.4) whose posterior percentiles form the comparison column for validating DE-MC results)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- DE-MC percentiles with N = 14 differ at most 0.02 from the true percentiles of log(xi) and phi
- For N = 21 and 70 the only allowed discrepancy is 0.1 at the 97.5% point of log(xi), which must disappear with a tenfold longer burn-in
- Comparison table is saved as percentiles_comparison.csv and includes a WinBUGS 1.4 reference column for each N
- Jump scale gamma = 2.38/sqrt(2d) is applied with gamma = 1 every 10th generation
- Chains are initialized from overdispersed states so the population is not confined to a single modal region

## Limitations

- The 97.5% percentile of log(xi) shows a 0.1 discrepancy for N = 21 and 70 unless burn-in is extended tenfold
- Large populations (e.g. N = 70) require more computer time to converge than small ones
- If the initial population covers just a single modal region, distant modes cannot be reached
- gamma may need to be decreased if blocks are strongly correlated, as in the nonlinear mixed-effects model where gamma needed to be decreased somewhat to get an optimal acceptance rate
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence

## Evidence

- [intro] overdispersed parallel chains: "we run multiple Markov chains, which are initialized from overdispersed states, in parallel and let the chains learn from each other"
- [intro] DE-MC jump construction: "takes the difference of vectors of two randomly chosen chains, multiplies the dif- ference with a factor γ and adds the result to the vector of the current chain"
- [discussion] default jump scale: "The suggested default value of γ = 2 . 38 / √ (2d) performed well in bock updating (Table 4)"
