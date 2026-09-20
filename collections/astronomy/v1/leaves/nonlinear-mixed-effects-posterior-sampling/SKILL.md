---
name: nonlinear-mixed-effects-posterior-sampling
description: Use when the target is a high-dimensional Bayesian posterior from a nonlinear
  mixed-effects (population pharmacokinetic) model whose blocks or correlated parameters
  defeat one-dimensional Gibbs/WinBUGS-style updates and closed-form conditionals.
when_to_use_negative:
- The model is a generalized linear model amenable to the one-dimensional updates
  that WinBUGS already supports — outside the GLM context WinBUGS is limited, but
  inside it this block DE-MC machinery is unnecessary
- The parameter vector cannot be sensibly partitioned into blocks or the initial population
  covers only a single modal region, since distant far-away modes cannot then be reached
- A Gibbs sampler is feasible because the conditional distribution of the parameter
  (e.g. gamma) is available in closed form and easy to sample from directly
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment in which the Theophylline posterior, 15-block partition, and binomial-crossover
    block DE-MC sampler are implemented and run
- name: remotes
  role: Used to install the demc2 sampler package from GitHub via remotes::install_github
- name: demc2
  role: R library providing Differential Evolution Markov Chain parameter estimation
    / adaptive MCMC on real parameter spaces for the block updates
  repo: https://github.com/CajoterBraak/demc2
- name: WinBUGS
  role: 'Contrast baseline: limited to one-dimensional updates outside the generalized
    linear model context, motivating the block DE-MC approach'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:08f1779369ea35c13e2516da681a711362cfa1d0e508d243d60e1b1cb95aa4f2
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
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# nonlinear-mixed-effects-posterior-sampling

## Summary

Samples the posterior of a 43-parameter nonlinear mixed-effects model (the Theophylline pharmacokinetic model) using block Differential Evolution Markov Chain (DE-MC), partitioning the parameter vector into 15 blocks and letting multiple chains in parallel learn from each other's differences.

## When to use

Use when the target is a high-dimensional Bayesian posterior from a nonlinear mixed-effects (population pharmacokinetic) model whose blocks or correlated parameters defeat one-dimensional Gibbs/WinBUGS-style updates and closed-form conditionals.

## When NOT to use

- The model is a generalized linear model amenable to the one-dimensional updates that WinBUGS already supports — outside the GLM context WinBUGS is limited, but inside it this block DE-MC machinery is unnecessary
- The parameter vector cannot be sensibly partitioned into blocks or the initial population covers only a single modal region, since distant far-away modes cannot then be reached
- A Gibbs sampler is feasible because the conditional distribution of the parameter (e.g. gamma) is available in closed form and easy to sample from directly

## Inputs

- Theophylline nonlinear mixed-effects posterior definition (likelihood and priors over 43 parameters: per-patient parameters, l Ke, l Ka, l Cl, tau-parameters, sigma2)
- Block partition specification (12 per-patient blocks + location block + tau block + sigma2 block = 15 blocks)
- Overdispersed initial states for N parallel chains
- Crossover probability CR

## Outputs

- Posterior samples per parameter block written to theophylline_block_demc_results.csv
- Convergence diagnostics based on mixing of overdispersed-start chains to the same target distribution
- Per-block acceptance rates used to tune gamma

## How to apply

Set up the Theophylline nonlinear mixed-effects posterior in R and partition its 43-dimensional parameter vector into 15 blocks: twelve per-patient blocks, a location block (l Ke, l Ka, l Cl), a tau block, and a sigma2 block. For each block, form the DE-MC jump as the difference of two randomly chosen chains among the remaining N-1 chains, scaled by the block-specific factor gamma = 2.38/sqrt(2*d_b), and apply binomial crossover with probability CR so only a random subset of block components receives the jump; accept or reject with a standard Metropolis step. Run two inner iterations of block updates per outer iteration with multiple chains initialized from overdispersed states running in parallel. Because the block gamma default 2.38/sqrt(2d) may need to be decreased when blocks are strongly correlated, tune gamma to keep the acceptance rate optimal (the nonlinear mixed-effects model required a somewhat decreased gamma). Confirm convergence by checking that all chains from overdispersed starts mix to the same target distribution, then write posterior samples and diagnostics to theophylline_block_demc_results.csv.

## Related tools

- **R** (Environment in which the Theophylline posterior, 15-block partition, and binomial-crossover block DE-MC sampler are implemented and run)
- **remotes** (Used to install the demc2 sampler package from GitHub via remotes::install_github)
- **demc2** (R library providing Differential Evolution Markov Chain parameter estimation / adaptive MCMC on real parameter spaces for the block updates) — https://github.com/CajoterBraak/demc2
- **WinBUGS** (Contrast baseline: limited to one-dimensional updates outside the generalized linear model context, motivating the block DE-MC approach)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- All chains initialized from overdispersed states converge to the same target distribution (cross-chain convergence check)
- Acceptance rate after gamma tuning matches the optimal range; the article notes gamma = 2.38/sqrt(2d) performed well in block updating but had to be decreased somewhat in the nonlinear mixed-effects model
- Block partition is complete and consistent: 12 per-patient blocks + location block (l Ke, l Ka, l Cl) + tau block + sigma2 block = 15 blocks covering all 43 parameters
- Each accepted/rejected proposal is a valid Metropolis step with each jump as likely as the reverse jump given the current state of the remaining chains
- Output CSV theophylline_block_demc_results.csv contains posterior samples and convergence diagnostics for all 43 parameters

## Limitations

- If blocks are strongly correlated, gamma may need to be decreased below 2.38/sqrt(2d) to maintain an optimal acceptance rate
- If the initial population covers just a single modal region, distant far-away modes cannot be reached
- DE-MC is not a greedy algorithm — slower convergence in terms of speed; large populations require more computer time to converge than small ones
- No theoretical analysis of the rate of convergence of DE-MC exists yet

## Evidence

- [methods] Block partition of the Theophylline parameter vector: [evidence span withheld — anchor status: not_found]
