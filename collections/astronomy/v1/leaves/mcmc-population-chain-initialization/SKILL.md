---
name: mcmc-population-chain-initialization
description: Use when sampling from a high-dimensional Bayesian posterior in real
  parameter space where choosing jump scale/orientation for standard MCMC is hard,
  and WinBUGS-style one-dimensional updates are too limited.
when_to_use_negative:
- The target distribution is multimodal and the initial population covers just a single
  modal region — distant modes cannot be reached
- A greedy, fast-converging optimizer is needed — DE-MC is not a greedy algorithm
  and large populations require more computer time to converge
- Gibbs sampling of a conditional is available in closed form and cheap — the DE approach
  is meant for cases where conditionals are hard to sample directly
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Runtime environment in which the demc2 package is installed and DE-MC chains
    are run
- name: remotes
  role: Used to install the demc2 package from GitHub via remotes::install_github
- name: TraitEnvMLMWA
  role: Released version installable from GitHub that carries the DE-MC (demc2) parameter
    estimation library
- name: WinBUGS
  role: Reference software for one-dimensional-update MCMC; its results serve as comparison
    (e.g. WinBUGS 1.4 percentile columns)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:d7a5d7d1069ec92a838a407db919ad310cbd192817e934f2cdf5e3ab52e29a3d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - TraitEnvMLMWA
  - WinBUGS
  - remotes
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mcmc-population-chain-initialization

## Summary

Initialize an overdispersed population of N Markov chains and run Differential Evolution Markov Chain (DE-MC), where each chain proposes jumps derived from the difference of two other randomly chosen chains scaled by gamma = 2.38/sqrt(2d), so chains learn from each other and jump scale/orientation adapt to the target's variance-covariance matrix.

## When to use

Use when sampling from a high-dimensional Bayesian posterior in real parameter space where choosing jump scale/orientation for standard MCMC is hard, and WinBUGS-style one-dimensional updates are too limited.

## When NOT to use

- The target distribution is multimodal and the initial population covers just a single modal region — distant modes cannot be reached
- A greedy, fast-converging optimizer is needed — DE-MC is not a greedy algorithm and large populations require more computer time to converge
- Gibbs sampling of a conditional is available in closed form and cheap — the DE approach is meant for cases where conditionals are hard to sample directly

## Inputs

- Bayesian model specification with prior hyperparameters (e.g. alpha=1, beta=10, a=1.85, b=0.1 for the one-way random-effects peak-discharge model)
- Population size N of chains (e.g. 14, 21, or 70) and overdispersed initial states per chain
- Jump scale gamma, default 2.38/sqrt(2d), with gamma = 1 every 10th generation

## Outputs

- Posterior draws of model parameters (e.g. log(xi) and phi) from each of the N chains
- Posterior percentiles of log(xi) and phi comparable to true values and WinBUGS 1.4 output (Table 2)
- Comparison table such as percentiles_comparison.csv

## How to apply

Set up the model (e.g. the Liu & Hodges peak-discharge one-way random-effects model with I=6 groups, J=4 observations per group, priors alpha=1, beta=10, a=1.85, b=0.1) and initialize an overdispersed population of N chains across the joint N x d-dimensional space. Run DE-MC, deriving each proposal by taking the difference of vectors of two randomly chosen chains among the remaining N-1 chains, multiplying by gamma, and adding to the current chain; use gamma = 2.38/sqrt(2d) as the default, with gamma = 1 every 10th generation (as in the Table 2 percentile comparison for N = 14, 21, 70). Each proposal defines a Metropolis step, so the N chains form a single random-walk Markov chain and the jumps automatically adapt to the variance-covariance matrix of the target distribution. Extract posterior draws (e.g. log(xi) and phi), compute percentiles, and compare against analytically known true values or WinBUGS 1.4 reference results.

## Related tools

- **R** (Runtime environment in which the demc2 package is installed and DE-MC chains are run)
- **remotes** (Used to install the demc2 package from GitHub via remotes::install_github)
- **TraitEnvMLMWA** (Released version installable from GitHub that carries the DE-MC (demc2) parameter estimation library)
- **WinBUGS** (Reference software for one-dimensional-update MCMC; its results serve as comparison (e.g. WinBUGS 1.4 percentile columns))

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- DE-MC percentiles of log(xi) and phi differ at most ~0.02 from analytically known true percentiles when N = 14 (Table 2 setup)
- Any larger discrepancy (e.g. the 0.1 difference at the 97.5% point of log(xi) for N = 21 and 70) disappears with a tenfold longer burn-in
- Acceptance behavior is consistent with a valid Metropolis step where each jump is as likely as the reverse jump given the current state of the remaining chains
- Empirical covariance of the sampled draws reflects the target's variance-covariance matrix, since DE-MC jump scale and orientation adapt automatically
- Results reproduce or fall within sampling error of the WinBUGS 1.4 reference column in Table 2

## Limitations

- No theoretical analysis of the rate of convergence of DE-MC exists yet; the authors state it is much desired
- If the initial population covers just a single modal region, there is no chance that other modes far away can be reached
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence; large populations require more computer time to converge than small ones
- gamma may need to be decreased if blocks are strongly correlated or to achieve an optimal acceptance rate (as in the nonlinear mixed-effects example)
- The conditional distribution required for Gibbs sampling of gamma will often not be available in closed form or easy to sample directly

## Evidence

- [intro] Jump derivation via difference of two random chains scaled by gamma: [evidence span withheld — anchor status: not_found]
