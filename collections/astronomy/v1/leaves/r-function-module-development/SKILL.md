---
name: r-function-module-development
description: Use when implementing or exporting a Bayesian MCMC sampler as a reusable
  R function module, specifically a Differential Evolution Markov Chain (DE-MC) sampler
  for real parameter spaces.
when_to_use_negative:
- The target requires block updates within a one-dimensional Gibbs-style framework
  such as WinBUGS, which outside the generalized linear model context is limited to
  one-dimensional updates
- The initial population can only cover a single modal region — distant modes of a
  multi-modal target cannot be reached, so a global exploration method with different
  initialization is needed
- 'A greedy, fast-converging optimizer is required: DE-MC is explicitly not a greedy
  algorithm, which is a disadvantage in terms of speed of convergence'
tools:
- name: R
  role: Language for implementing the ~10-line DE-MC core loop and exporting it as
    de_mc_sampler.R
  repo: CajoterBraak/demc2
- name: remotes
  role: Used to distribute/install the R function module from GitHub via remotes::install_github
  repo: CajoterBraak/demc2
- name: demc2
  role: Published R library version of Adaptive MCMC on real parameter spaces via
    Differential Evolution Markov Chain, showing the module distribution pattern
  repo: CajoterBraak/demc2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:53c899e74592b110fdcf1e77829d14858a7e6f62ddd76b466c58a8c0ed9dc331
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - demc2
  - remotes
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# r-function-module-development

## Summary

Wraps the DE-MC algorithm of ter Braak (2006) into a compact R function module (~10-line core, de_mc_sampler.R) that samples an N x d target space via the proposal x_p = x_i + gamma*(x_R1 - x_R2) + e with Metropolis acceptance.

## When to use

Use when implementing or exporting a Bayesian MCMC sampler as a reusable R function module, specifically a Differential Evolution Markov Chain (DE-MC) sampler for real parameter spaces.

## When NOT to use

- The target requires block updates within a one-dimensional Gibbs-style framework such as WinBUGS, which outside the generalized linear model context is limited to one-dimensional updates
- The initial population can only cover a single modal region — distant modes of a multi-modal target cannot be reached, so a global exploration method with different initialization is needed
- A greedy, fast-converging optimizer is required: DE-MC is explicitly not a greedy algorithm, which is a disadvantage in terms of speed of convergence

## Inputs

- Target (unnormalized) density function pi over a d-dimensional real parameter space
- Initial population of N chains drawn from overdispersed starting states (N x d matrix)
- Scaling factor gamma (default 2.38/sqrt(2d))
- Perturbation distribution parameters (e.g. e ~ N(0, b*I) with b small)

## Outputs

- R function module de_mc_sampler.R implementing the DE-MC update loop
- N x d matrix of posterior samples of the parallel Markov chains

## How to apply

Initialize a population of N chains from overdispersed starting states in the d-dimensional target real parameter space. For each chain i, draw two chains R1 and R2 at random from the remaining N-1 chains and compute the proposal x_p = x_i + gamma*(x_R1 - x_R2) + e, where e ~ N(0, b*I) with small b is a symmetric perturbation with unbounded support so the whole parameter space can be reached. Use the default scaling gamma = 2.38/sqrt(2d) so the difference vector carries the scale and orientation of the target's variance-covariance matrix. Accept with probability min(1, pi(x_p)/pi(x_i)), a Metropolis step in which each jump is as likely as the reverse jump given the current state of the remaining chains. Wrap the update loop in an R function (~10 lines for the core) exported as de_mc_sampler.R, then distribute it as a package installable via install.packages("remotes") and remotes::install_github, as the demc2 R library does.

## Related tools

- **R** (Language for implementing the ~10-line DE-MC core loop and exporting it as de_mc_sampler.R) — CajoterBraak/demc2
- **remotes** (Used to distribute/install the R function module from GitHub via remotes::install_github) — CajoterBraak/demc2
- **demc2** (Published R library version of Adaptive MCMC on real parameter spaces via Differential Evolution Markov Chain, showing the module distribution pattern) — CajoterBraak/demc2

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- The module runs without error on a simple test target
- Each proposal satisfies the symmetric-jump Metropolis criterion: each jump is as likely as the reverse jump given the current state of the remaining chains
- The core update loop stays within about 10 lines of R, matching the paper's DE-MC specification
- Acceptance probability equals min(1, pi(x_p)/pi(x_i)) for every proposal
- With the default gamma = 2.38/sqrt(2d) the acceptance rate is near-optimal; gamma may need to be decreased for strongly correlated blocks

## Limitations

- No theoretical analysis of the rate of convergence of DE-MC exists yet; the authors state it is much desired
- DE-MC is not a greedy algorithm, a disadvantage in terms of speed of convergence; finding greedier variants that retain robustness and simplicity is an open challenge
- If the initial population covers just a single modal region, there is no chance that other modes that are far away can be reached
- Large populations require more computer time to converge than small ones; gamma may need to be decreased if blocks are strongly correlated or to get an optimal acceptance rate

## Evidence

- [intro] The core of the method can be coded in about 10 lines: [evidence span withheld — anchor status: not_found]
