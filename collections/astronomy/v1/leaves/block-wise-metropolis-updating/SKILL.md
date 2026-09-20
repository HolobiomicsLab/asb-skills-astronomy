---
name: block-wise-metropolis-updating
description: Use when a multi-parameter Bayesian posterior (e.g. nonlinear mixed-effects
  models like the 43-parameter Theophylline model) mixes poorly under single-parameter
  Gibbs or one-dimensional updates such as those WinBUGS performs outside GLMs.
when_to_use_negative:
- When a closed-form conditional distribution is available and easy to sample directly,
  plain Gibbs sampling of that block is simpler than block-wise DE-MC Metropolis steps
- When only the initial population covers a single modal region and distant modes
  must be reached — DE-MC cannot jump there regardless of blocking
- When the problem is a generalized linear model already handled well by WinBUGS one-dimensional
  updates
edam_operation: http://edamontology.org/operation_3722
edam_topics:
- http://edamontology.org/topic_0167
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Implementation environment for the block DE-MC sampler over the Theophylline
    posterior
- name: remotes
  role: Installed via install.packages('remotes') to fetch the demc2 sampler from
    GitHub
- name: demc2
  role: R library providing Differential Evolution Markov Chain parameter estimation
    for Adaptive MCMC on real parameter spaces, used to run the block updates
  repo: https://github.com/CajoterBraak/demc2
- name: WinBUGS
  role: 'Contrast target: limited to one-dimensional updates outside the generalized
    linear model context, motivating block-wise updating'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:b0e7d782f216a52d03ad8619386a404400a4639aa0044a8ebafe450f25f86b00
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
  edam_operation: http://edamontology.org/operation_3722
  edam_topics:
  - http://edamontology.org/topic_0167
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# block-wise-metropolis-updating

## Summary

Partition a high-dimensional Bayesian posterior parameter vector into blocks (e.g. the 15-block split of the Theophylline nonlinear mixed-effects model into 12 per-patient blocks plus location, tau, and sigma2 blocks) and update each block with a Metropolis step using DE-MC jumps, so sampling proceeds in lower-dimensional spaces.

## When to use

Use when a multi-parameter Bayesian posterior (e.g. nonlinear mixed-effects models like the 43-parameter Theophylline model) mixes poorly under single-parameter Gibbs or one-dimensional updates such as those WinBUGS performs outside GLMs.

## When NOT to use

- When a closed-form conditional distribution is available and easy to sample directly, plain Gibbs sampling of that block is simpler than block-wise DE-MC Metropolis steps
- When only the initial population covers a single modal region and distant modes must be reached — DE-MC cannot jump there regardless of blocking
- When the problem is a generalized linear model already handled well by WinBUGS one-dimensional updates

## Inputs

- Log-posterior function of a nonlinear mixed-effects model (e.g. Theophylline with 43 parameters: 12 per-patient blocks, a location block, a tau block, and a sigma2 block)
- Multiple MCMC chains initialized from overdispersed states
- Block partition of the parameter vector with per-block dimension d_b
- Crossover probability CR and block-specific scaling factor gamma = 2.38/sqrt(2*d_b)

## Outputs

- Posterior samples per parameter block written to theophylline_block_demc_results.csv
- Convergence diagnostics confirming all chains mix to the same target distribution

## How to apply

Partition the parameter vector into blocks by structure: per-patient blocks plus shared population blocks (location parameters l Ke, l Ka, l Cl; the tau-parameters; and sigma2 alone formed the final block in the Theophylline case, giving 15 blocks total). For each block of dimension d_b, form the DE-MC jump as the difference of vectors of two randomly chosen chains among the remaining N-1 chains, scaled by gamma = 2.38/sqrt(2*d_b), and apply binomial crossover with probability CR so only a random subset of block components receives the jump. Accept or reject each block proposal with a standard Metropolis step; run two inner iterations of block updates per outer iteration with multiple chains initialized from overdispersed states. Convergence is judged by confirming that all overdispersed-start chains mix to the same target distribution; decrease gamma if blocks are strongly correlated, as was needed for the nonlinear mixed-effects model to reach an optimal acceptance rate.

## Related tools

- **R** (Implementation environment for the block DE-MC sampler over the Theophylline posterior)
- **remotes** (Installed via install.packages('remotes') to fetch the demc2 sampler from GitHub)
- **demc2** (R library providing Differential Evolution Markov Chain parameter estimation for Adaptive MCMC on real parameter spaces, used to run the block updates) — https://github.com/CajoterBraak/demc2
- **WinBUGS** (Contrast target: limited to one-dimensional updates outside the generalized linear model context, motivating block-wise updating)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- The 43-parameter vector is split into exactly 15 blocks: 12 per-patient blocks plus the l Ke/l Ka/l Cl location block, the tau-parameters block, and a sigma2-only block
- Block-specific scaling uses gamma = 2.38/sqrt(2*d_b), which the paper reports performed well in block updating (Table 4)
- Chains started from overdispersed states all converge to the same target distribution
- Acceptance rate is near-optimal after tuning gamma (it needed to be decreased somewhat in the nonlinear mixed-effects model)
- Two inner iterations of block updates are executed per outer iteration before the next outer step

## Limitations

- If blocks are strongly correlated, gamma may need to be decreased below the 2.38/sqrt(2d) default
- If the initial population covers just a single modal region, distant modes cannot be reached
- Large populations require more computer time to converge than small ones
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence
- No theoretical analysis of the rate of convergence of DE-MC exists yet

## Evidence

- [results] 15-block partition of the Theophylline model: [evidence span withheld — anchor status: not_found]
