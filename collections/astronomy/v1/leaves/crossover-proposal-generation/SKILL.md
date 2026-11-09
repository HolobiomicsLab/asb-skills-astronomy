---
name: crossover-proposal-generation
description: Use when sampling a high-dimensional posterior (e.g. the 43-parameter
  Theophylline nonlinear mixed-effects model) via block updating, where full-vector
  DE-MC jumps mix poorly and per-block jumps with binomial crossover are needed.
when_to_use_negative:
- The target posterior is low-dimensional or an uncorrelated full-vector update already
  mixes well — plain DE-MC without block partitioning is simpler (DE-MC core can be
  coded in about 10 lines).
- The initial chain population covers only a single modal region and distant modes
  exist — crossover/block updates will not reach far-away modes.
- A Gibbs sampler is possible because the conditional distribution of the parameter
  (e.g. γ) is available in closed form and easy to sample directly.
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3053
tools:
- name: demc2
  role: R package implementing Differential Evolution Markov Chain with block updating
    and crossover proposals for adaptive MCMC on real parameter spaces
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: Used to install the demc2 sampler package from GitHub via remotes::install_github
- name: R
  role: Environment in which the DE-MC block/crossover proposals are coded and executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:8a584fb9acf95d3892bfe48e9a47c6845095e1665822e4133fc4298971dce687
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - TraitEnvMLMWA
  - demc2
  - remotes
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# crossover-proposal-generation

## Summary

Generates block-wise DE-MC proposals by combining a differential-evolution jump (difference of two randomly chosen chains, scaled by γ = 2.38/sqrt(2*d_b)) with binomial crossover so only a random subset of block components receives the jump, enabling lower-dimensional Metropolis updates within a high-dimensional posterior.

## When to use

Use when sampling a high-dimensional posterior (e.g. the 43-parameter Theophylline nonlinear mixed-effects model) via block updating, where full-vector DE-MC jumps mix poorly and per-block jumps with binomial crossover are needed.

## When NOT to use

- The target posterior is low-dimensional or an uncorrelated full-vector update already mixes well — plain DE-MC without block partitioning is simpler (DE-MC core can be coded in about 10 lines).
- The initial chain population covers only a single modal region and distant modes exist — crossover/block updates will not reach far-away modes.
- A Gibbs sampler is possible because the conditional distribution of the parameter (e.g. γ) is available in closed form and easy to sample directly.

## Inputs

- Posterior log-density function of the target parameter vector (e.g. the Theophylline nonlinear mixed-effects model, 43 parameters)
- Block partition of the parameter vector (e.g. 15 blocks: 12 per-patient blocks, a location block of l Ke/l Ka/l Cl, a tau block, a sigma2 block)
- Population of multiple Markov chains initialized from overdispersed states
- Crossover probability CR and block dimension d_b for the γ = 2.38/sqrt(2*d_b) scaling

## Outputs

- Posterior samples per chain after block-wise crossover proposal and Metropolis accept/reject steps
- Chain mixing/convergence diagnostics from overdispersed starts (e.g. written to theophylline_block_demc_results.csv)

## How to apply

Partition the parameter vector into blocks (for the Theophylline model: a location block of l Ke, l Ka, l Cl, a tau block, a sigma2 block, plus twelve per-patient blocks, giving 15 blocks). For each block of dimension d_b, form the DE-MC jump as the difference of vectors of two randomly chosen chains among the remaining N-1 chains, scaled by the block-specific factor γ = 2.38/sqrt(2*d_b), since the difference vector automatically carries scale and orientation information from the target's variance-covariance matrix. Apply binomial crossover with crossover probability CR so only a random subset of the block's components receives the jump. Accept or reject via a standard Metropolis step; run two inner block updates per outer iteration, with chains initialized from overdispersed states. If blocks are strongly correlated, decrease γ to maintain an acceptable acceptance rate.

## Related tools

- **demc2** (R package implementing Differential Evolution Markov Chain with block updating and crossover proposals for adaptive MCMC on real parameter spaces) — https://github.com/CajoterBraak/demc2
- **remotes** (Used to install the demc2 sampler package from GitHub via remotes::install_github)
- **R** (Environment in which the DE-MC block/crossover proposals are coded and executed)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Acceptance rate of the Metropolis steps falls in a reasonable range; γ may need to be decreased somewhat if it is suboptimal (as in the nonlinear mixed-effects model).
- All chains started from overdispersed states converge to and mix over the same target distribution (convergence by cross-chain agreement rather than independent-run comparison).
- Each accepted proposal's jump is as likely as the reverse jump given the current state of the remaining chains — the proposal must define a valid Metropolis step on the N×d-dimensional space.
- With γ = 2.38/sqrt(2*d_b) the block updates mix well (performed well in block updating, Table 4); if blocks are strongly correlated, verify mixing improves after decreasing γ.

## Limitations

- If blocks are strongly correlated, γ = 2.38/sqrt(2*d_b) may need to be decreased; in the nonlinear mixed-effects model γ needed to be decreased somewhat to get an optimal acceptance rate.
- No theoretical analysis of the rate of convergence of DE-MC exists yet; it is much desired.
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence, and finding greedier variants that retain robustness and simplicity is an open challenge.
- Large populations require more computer time to converge than small ones.

## Evidence
