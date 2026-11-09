---
name: population-based-multi-chain-mcmc
description: Use when sampling from a high-dimensional real-parameter posterior (e.g.
  Bayesian analysis) where choosing scale/orientation for a jumping distribution is
  hard, or WinBUGS one-dimensional updates are insufficient.
when_to_use_negative:
- When a closed-form Gibbs step exists — e.g. the conditional distribution needed
  for Gibbs sampling of a parameter is available in closed form and easy to sample
  directly
- When the initial population covers only a single modal region, since distant modes
  cannot be reached by the sampler
- When greedy, fast-converging optimization-like sampling is required, since DE-MC
  is not a greedy algorithm and large populations need more computer time to converge
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Implementation language for the ~10-line DE-MC core sampler (de_mc_sampler.R)
    and the demc2 adaptive MCMC library
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: Installation vehicle for the demc2 package from GitHub
  repo: https://github.com/CajoterBraak/demc2
- name: demc2
  role: R library providing Differential Evolution Markov Chain adaptive MCMC on real
    parameter spaces
  repo: https://github.com/CajoterBraak/demc2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:43838ca7c3be04acc3f12271d47b48e7907ff3495f0dc84e087f11e61ba3fc2a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - demc2
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# population-based-multi-chain-mcmc

## Summary

Implements DE-MC, a population-based Metropolis sampler in which each of N parallel chains proposes jumps as gamma*(x_R1 - x_R2) differences drawn from the other N-1 chains, so jump scale and orientation adapt automatically to the target's variance-covariance matrix.

## When to use

Use when sampling from a high-dimensional real-parameter posterior (e.g. Bayesian analysis) where choosing scale/orientation for a jumping distribution is hard, or WinBUGS one-dimensional updates are insufficient.

## When NOT to use

- When a closed-form Gibbs step exists — e.g. the conditional distribution needed for Gibbs sampling of a parameter is available in closed form and easy to sample directly
- When the initial population covers only a single modal region, since distant modes cannot be reached by the sampler
- When greedy, fast-converging optimization-like sampling is required, since DE-MC is not a greedy algorithm and large populations need more computer time to converge

## Inputs

- A real-parameter target distribution, typically an unnormalised posterior density pi(x) of dimension d from a Bayesian model
- A population size N of overdispersed initial chain states
- Scaling factor gamma (default 2.38/sqrt(2d)) and small symmetric perturbation distribution e (e.g. N(0, b*I) with small b)

## Outputs

- N parallel chains of posterior samples over the d-dimensional parameter space
- Posterior estimates of real-valued model parameters with converged multi-chain diagnostics

## How to apply

Initialize a population of N chains from overdispersed starting states in the d-dimensional target space. For each chain i, draw two chains R1 and R2 at random from the remaining N-1 chains and compute the proposal x_p = x_i + gamma*(x_R1 - x_R2) + e, with e a small symmetric perturbation (e.g. N(0, b*I) with unbounded support) so each jump is as likely as its reverse. Accept with probability min(1, pi(x_p)/pi(x_i)), which defines a Metropolis step on the joint N x d space. Use the default scaling gamma = 2.38/sqrt(2d), decreasing it when blocks are strongly correlated or to reach an optimal acceptance rate. The core sampler is about 10 lines of R; the demc2 package implements the adaptive version.

## Related tools

- **R** (Implementation language for the ~10-line DE-MC core sampler (de_mc_sampler.R) and the demc2 adaptive MCMC library) — https://github.com/CajoterBraak/demc2
- **remotes** (Installation vehicle for the demc2 package from GitHub) — https://github.com/CajoterBraak/demc2
- **demc2** (R library providing Differential Evolution Markov Chain adaptive MCMC on real parameter spaces) — https://github.com/CajoterBraak/demc2

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Each proposal satisfies the symmetric-jump Metropolis criterion: each jump is as likely as the reverse jump given the current state of the remaining chains
- The module runs without error on a simple test target (e.g. a known normal or Student distribution)
- Default gamma = 2.38/sqrt(2d) yields a good acceptance rate; if acceptance is suboptimal, decreasing gamma (as in the nonlinear mixed-effects model) improves it
- Multiple chains initialized from overdispersed states agree, indicating convergence as in a single random-walk Markov chain on the N x d space

## Limitations

- If the initial population covers just a single modal region, there is no chance that other modes far away can be reached
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence; large populations require more computer time to converge than small ones
- No theoretical analysis of the rate of convergence of DE-MC exists yet
- gamma may need to be decreased if blocks are strongly correlated, or to obtain an optimal acceptance rate

## Evidence
