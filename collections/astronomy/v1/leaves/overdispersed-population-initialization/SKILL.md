---
name: overdispersed-population-initialization
description: Use when setting up a multi-chain DE-MC (or similar population MCMC)
  run and you need starting populations that are wide enough for R-hat-based convergence
  diagnosis to be informative.
when_to_use_negative:
- When the initial population is intended to cover only a single known modal region
  — distant modes cannot be reached, so overdispersion across modes is pointless or
  misleading
- When running a single independent chain with no Gelman-Rubin convergence checking
  (e.g. a WinBUGS-style single-chain update), where overdispersed populations serve
  no diagnostic purpose
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: demc2
  role: R package implementing Differential Evolution Markov Chain whose parallel
    chains are initialized from the overdispersed population
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: R package used to install demc2 from GitHub before constructing the initial
    populations
- name: R
  role: Environment in which the initial populations are generated and DE-MC is run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:451e72b6f5e4451ba53209a202608f196217455cd43f6a3c6649ca955e32396c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - TraitEnvMLMWA
  - demc2
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# overdispersed-population-initialization

## Summary

Initialize the parallel DE-MC chains from deliberately overdispersed starting states so that Gelman-Rubin R-hat convergence monitoring is meaningful and the population covers the target region. This mirrors the paper's practice of running multiple chains initialized from overdispersed states that learn from each other.

## When to use

Use when setting up a multi-chain DE-MC (or similar population MCMC) run and you need starting populations that are wide enough for R-hat-based convergence diagnosis to be informative.

## When NOT to use

- When the initial population is intended to cover only a single known modal region — distant modes cannot be reached, so overdispersion across modes is pointless or misleading
- When running a single independent chain with no Gelman-Rubin convergence checking (e.g. a WinBUGS-style single-chain update), where overdispersed populations serve no diagnostic purpose

## Inputs

- Target log-density definition (e.g. d=100 standard Normal)
- Population size N (e.g. 101, 200 or 1000 chains)
- Initial spread specification for narrow vs. broad/overdispersed starting populations

## Outputs

- Initial population matrix (N chains x d parameters) per starting state
- R-hat trace usable for convergence monitoring across the overdispersed chains

## How to apply

Generate two initial populations per population size N (e.g. N = 101, 200, 1000): one narrow starting state and one broad/overdispersed starting state, so the overdispersed set serves as the parallel chains for the Gelman-Rubin diagnostic while the population learns from each other. The N chains form a single random walk on the N x d-dimensional space, so spread in the initial population directly controls both convergence monitoring sensitivity and coverage of the target's mode(s). For a d=100 standard Normal target this initialization supported tracking acceptance fraction (converging to approximately 0.23) and R-hat (first dropping below 1.2 around 900-1000 generations). Note the paper's warning: if the initial population covers just a single modal region, distant modes cannot be reached, so spread should reflect any uncertainty about which mode contains the posterior mass.

## Related tools

- **demc2** (R package implementing Differential Evolution Markov Chain whose parallel chains are initialized from the overdispersed population) — https://github.com/CajoterBraak/demc2
- **remotes** (R package used to install demc2 from GitHub before constructing the initial populations)
- **R** (Environment in which the initial populations are generated and DE-MC is run)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Two distinct initial populations exist per N (narrow and broad/overdispersed) as specified in the workflow
- Gelman-Rubin R-hat computed across the overdispersed chains first drops below 1.2 around 900-1000 generations for the d=100 Normal target
- Metropolis acceptance fraction across the N chains converges to approximately 0.23
- Generations-to-convergence matches expected values (~4000, 1000, 600 for N = 101, 200, 1000), confirming the overdispersed start did not stall convergence
- Acceptance/R-hat traces and convergence summary are written to demc_d100_normal_results.csv

## Limitations

- If the initial population covers just a single modal region, there is no chance that other modes far away can be reached
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence; large populations require more computer time to converge than small ones
- No theoretical analysis of the rate of convergence of DE-MC exists yet

## Evidence

- [methods] Define the d=100 standard Normal log-density and generate two initial populations per N (narrow and broad/overdispersed starting states for the parallel chains).: [evidence span withheld — anchor status: not_found]
- [methods] At each generation, compute the Gelman-Rubin R-hat statistic across the overdispersed chains and record the generation at which R-hat first drops below 1.2: [evidence span withheld — anchor status: not_found]
