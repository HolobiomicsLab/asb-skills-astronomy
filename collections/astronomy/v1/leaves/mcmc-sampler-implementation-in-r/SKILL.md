---
name: mcmc-sampler-implementation-in-r
description: Use when Bayesian computing requires sampling a real-parameter target
  density (e.g. multivariate Normal or Student t3 posteriors) and scale/orientation
  of the jumping distribution must adapt automatically, or when benchmarking MCMC
  efficiency vs RWM.
when_to_use_negative:
- Blocks of parameters are strongly correlated — gamma = 2.38/sqrt(2d) may need to
  be decreased, so the default scale is not automatically appropriate
- The initial population covers only a single modal region — distant modes cannot
  be reached, so DE-MC is the wrong choice for widely separated multimodal targets
  without adjusted initialization
- The model requires one-dimensional updates in a generalized linear model context
  where WinBUGS already suffices
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3570
tools:
- name: R
  role: Implementation language in which the DE-MC sampler and target densities are
    coded and run
- name: remotes
  role: R package used to install demc2 from GitHub via remotes::install_github("CajoterBraak/demc2")
- name: demc2
  role: R library providing Differential Evolution Markov Chain adaptive MCMC on real
    parameter spaces
  repo: https://github.com/CajoterBraak/demc2
- name: TraitEnvMLMWA
  role: R package whose README install instructions (install.packages("remotes");
    remotes::install_github(...)) document the installation workflow for demc2
  repo: https://github.com/CajoterBraak/demc2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:1a0ec33af90b2861e328f4b84120e38227ba188e367ed949d12dd7506a361c7b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - TraitEnvMLMWA
  - demc2
  - remotes
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3570
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# mcmc-sampler-implementation-in-r

## Summary

Implement and run the Differential Evolution Markov Chain (DE-MC) sampler in R via the demc2 package, using multi-chain jumps derived from differences of randomly chosen chains with jump scale gamma = 2.38/sqrt(2d), to sample real-parameter posterior distributions and benchmark efficiency against random walk Metropolis.

## When to use

Use when Bayesian computing requires sampling a real-parameter target density (e.g. multivariate Normal or Student t3 posteriors) and scale/orientation of the jumping distribution must adapt automatically, or when benchmarking MCMC efficiency vs RWM.

## When NOT to use

- Blocks of parameters are strongly correlated — gamma = 2.38/sqrt(2d) may need to be decreased, so the default scale is not automatically appropriate
- The initial population covers only a single modal region — distant modes cannot be reached, so DE-MC is the wrong choice for widely separated multimodal targets without adjusted initialization
- The model requires one-dimensional updates in a generalized linear model context where WinBUGS already suffices

## Inputs

- Target log-density function defined in R (e.g. multivariate Normal or Student t3 with unit variances and pairwise correlations of 0.5)
- Dimensionality d (e.g. 5, 50, 100) and population size N (e.g. 2d, 3d, 10d)
- Overdispersed initial states for the N parallel chains

## Outputs

- Posterior draws from the N-chain DE-MC sampler for each (d, N) configuration
- Efficiency table (e.g. efficiency_table.csv) of ratios 100 x MSE_RWM / MSE_DE-MC for P50 and pooled P2.5 percentile estimates, spanning the reported 68%-501% range

## How to apply

Install remotes with install.packages("remotes"), then install the demc2 package from GitHub via remotes::install_github("CajoterBraak/demc2"). Define the target density (e.g. multivariate Normal or Student t3 with unit variances and pairwise correlations of 0.5) and initialize N parallel chains from overdispersed states. At each Metropolis step, draw a jump as gamma times the difference of two randomly chosen chains among the remaining N-1 chains, using the default jump scale gamma = 2.38/sqrt(2d); this lets jump scale and orientation adapt automatically to the variance-covariance matrix of the target. Run configurations such as N = 2d, 3d, 10d across dimensions d = 5, 50, 100 alongside a single-chain random walk Metropolis reference, collect posterior draws, and compute MSE of percentile estimates (e.g. P50 and pooled P2.5) across replicates to tabulate efficiency ratios 100 x MSE_RWM / MSE_DE-MC.

## Related tools

- **R** (Implementation language in which the DE-MC sampler and target densities are coded and run)
- **remotes** (R package used to install demc2 from GitHub via remotes::install_github("CajoterBraak/demc2"))
- **demc2** (R library providing Differential Evolution Markov Chain adaptive MCMC on real parameter spaces) — https://github.com/CajoterBraak/demc2
- **TraitEnvMLMWA** (R package whose README install instructions (install.packages("remotes"); remotes::install_github(...)) document the installation workflow for demc2) — https://github.com/CajoterBraak/demc2

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Computed efficiency ratios 100 x MSE_RWM / MSE_DE-MC fall within the reported 68%-501% range across the (d, N) grid for both Normal and Student t3 targets
- Efficiency increases as N/d grows (N = 2d, 3d, 10d) for fixed dimension d
- Pooled P2.5/97.5 percentile estimates show higher efficiency than median (P50) estimates
- Default gamma = 2.38/sqrt(2d) yields acceptable Metropolis acceptance rates without manual tuning for block updates
- Each proposal defines a valid Metropolis step in which each jump is as likely as the reverse jump given the current state of the remaining chains

## Limitations

- No theoretical analysis of the rate of convergence of DE-MC exists yet; the authors state it is much desired
- If the initial population covers just a single modal region, distant modes cannot be reached
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence
- Large populations require more computer time to converge than small ones
- gamma may need to be decreased if blocks are strongly correlated, and in a nonlinear mixed-effects model gamma needed to be decreased somewhat to get an optimal acceptance rate
- The conditional distribution required for Gibbs sampling gamma is often not available in closed form or not easy to sample from directly

## Evidence
