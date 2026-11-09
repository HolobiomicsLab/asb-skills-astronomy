---
name: differential-evolution-proposal-tuning
description: Use when fitting Bayesian models on real parameter spaces where an appropriate
  scale and orientation for the jumping distribution is hard to choose, e.g. nonlinear
  mixed-effects models, and multi-chain MCMC should adapt jumps automatically via
  gamma = 2.38/sqrt(2d).
when_to_use_negative:
- The initial population covers only a single modal region — distant modes of a multimodal
  target cannot be reached, so DE-MC is the wrong choice for that exploration problem
- The target is restricted to generalized linear models where one-dimensional (e.g.
  Gibbs/WinBUGS) updates are natural and sufficient
- The problem requires a greedier, faster-converging sampler — plain DE-MC is deliberately
  not greedy, so convergence speed suffers
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Language used to implement the ~10-line DE-MC proposal/acceptance kernel and
    to drive the multi-chain sampler
- name: demc2
  role: R library for Adaptive MCMC on real parameter spaces via Differential Evolution
    Markov Chain, providing the DE-MC kernel used here
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: Used to install demc2 from GitHub for running the tuned proposal kernel
- name: WinBUGS
  role: Reference one-dimensional-update MCMC sampler against which DE-MC proposal
    tuning (posterior quantile RMSE and runtime) is compared
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:9959142ab63006a2240c7d52e55f022b7f15852dbaf109a8af4b599991ff6bd1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - WinBUGS
  - demc2
  - remotes
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# differential-evolution-proposal-tuning

## Summary

Constructs the DE-MC proposal kernel x_p = x_i + gamma*(x_R1 - x_R2) + e from ter Braak (2006), so that jump scale and orientation are learned automatically from the population of chains instead of hand-tuned, with default gamma = 2.38/sqrt(2d) and a small symmetric perturbation e.

## When to use

Use when fitting Bayesian models on real parameter spaces where an appropriate scale and orientation for the jumping distribution is hard to choose, e.g. nonlinear mixed-effects models, and multi-chain MCMC should adapt jumps automatically via gamma = 2.38/sqrt(2d).

## When NOT to use

- The initial population covers only a single modal region — distant modes of a multimodal target cannot be reached, so DE-MC is the wrong choice for that exploration problem
- The target is restricted to generalized linear models where one-dimensional (e.g. Gibbs/WinBUGS) updates are natural and sufficient
- The problem requires a greedier, faster-converging sampler — plain DE-MC is deliberately not greedy, so convergence speed suffers

## Inputs

- Bayesian target density pi(x) on a real d-dimensional parameter space (e.g. the seven-parameter Theophylline nonlinear mixed-effects model)
- A population of N chains initialized from overdispersed starting states
- Jump scale gamma (default 2.38/sqrt(2d)) and perturbation variance b for e ~ N(0, b*I)

## Outputs

- DE-MC proposal kernel as an R module (~10 lines core, de_mc_sampler.R)
- Posterior samples from N parallel chains after burn-in
- Posterior quantile estimates (P2.5, P50, P97.5) per parameter, comparable against reference posterior values and WinBUGS in rmse_table.csv

## How to apply

Initialize a population of N chains from overdispersed states in the d-dimensional target space. For each chain i, draw two chains R1 and R2 at random from the remaining N-1 chains and compute the proposal x_p = x_i + gamma*(x_R1 - x_R2) + e, where e is drawn from a symmetric distribution with small variance but unbounded support (e.g. N(0, b*I) with small b) so the whole parameter space remains reachable; the difference vector (x_R1 - x_R2) carries scale and orientation information of the target. Accept with the Metropolis probability min(1, pi(x_p)/pi(x_i)); the default jump scale gamma = 2.38/sqrt(2d) is well justified by theory and performed well in block updating. Decision point: if blocks are strongly correlated (as in the nonlinear mixed-effects model), decrease gamma somewhat to reach an optimal acceptance rate. The core update loop can be implemented in about 10 lines of R (exported as de_mc_sampler.R), or used via the demc2 package installed from GitHub.

## Related tools

- **R** (Language used to implement the ~10-line DE-MC proposal/acceptance kernel and to drive the multi-chain sampler)
- **demc2** (R library for Adaptive MCMC on real parameter spaces via Differential Evolution Markov Chain, providing the DE-MC kernel used here) — https://github.com/CajoterBraak/demc2
- **remotes** (Used to install demc2 from GitHub for running the tuned proposal kernel)
- **WinBUGS** (Reference one-dimensional-update MCMC sampler against which DE-MC proposal tuning (posterior quantile RMSE and runtime) is compared)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Every proposal satisfies the symmetric-jump Metropolis criterion: each jump is as likely as the reverse jump given the current state of the remaining chains
- The N parallel chains together behave as a single random-walk Markov chain on the N x d-dimensional space (joint-state stationarity)
- Acceptance rate reaches its optimal range; gamma is reduced when blocks are strongly correlated, as needed in the nonlinear mixed-effects model
- After discarding 20% burn-in, maximum R-hat across parameters is below 1.2 across chains
- RMSE of posterior quantile estimates (P2.5, P50, P97.5) is comparable to or up to a factor of 4 lower than WinBUGS for location parameters (l Ke, l Ka, l Cl)

## Limitations

- If the initial population covers just a single modal region, distant modes cannot be reached by the proposal kernel
- Large populations require more computer time to converge than small ones
- gamma must be decreased somewhat when blocks are strongly correlated to obtain an optimal acceptance rate
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence; no theoretical analysis of the rate of convergence exists yet
- The random perturbation e must have small variance relative to the target but unbounded support, otherwise the whole parameter space is not reachable

## Evidence
