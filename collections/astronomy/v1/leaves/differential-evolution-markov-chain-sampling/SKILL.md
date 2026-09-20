---
name: differential-evolution-markov-chain-sampling
description: Use when sampling a Bayesian posterior over real-valued parameters (d
  ≈ 5–100+) where choosing jump scale/orientation for single-chain RWM is hard, or
  when benchmarking MCMC efficiency against WinBUGS or optimal-normal-jump random
  walk Metropolis.
when_to_use_negative:
- Discrete or non-real-parameter spaces — the difference-vector jump construction
  is defined over real parameter spaces only
- Posteriors with multiple distant modes when the initial population covers just a
  single modal region — far-away modes cannot be reached
- Problems needing a greedily fast sampler — DE-MC is explicitly not a greedy algorithm,
  a disadvantage in speed of convergence
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: demc2
  role: R package implementing DE-MC (and snooker updater variant) for adaptive MCMC
    on real parameter spaces; installed from GitHub
  repo: CajoterBraak/demc2
- name: remotes
  role: R package used to install demc2 from GitHub via remotes::install_github
- name: R
  role: Runtime environment in which DE-MC runs, target densities are defined, and
    percentile/efficiency tables are computed
- name: WinBUGS
  role: Reference sampler providing very-long-run posterior percentiles against which
    DE-MC results are compared
- name: TraitEnvMLMWA
  role: R package referenced alongside demc2 in the installation instructions of the
    associated repository
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  - task_008
  - task_003
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:d96564bed36aa0cae7d28b2c3857715daf7365465e9985bf27902b7e9a53d9e6
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# differential-evolution-markov-chain-sampling

## Summary

Differential Evolution Markov Chain (DE-MC) is a multi-chain MCMC method that proposes jumps as γ times the difference of two randomly chosen other chains, letting scale and orientation adapt automatically to the target's variance-covariance matrix for Bayesian computing in real parameter spaces.

## When to use

Use when sampling a Bayesian posterior over real-valued parameters (d ≈ 5–100+) where choosing jump scale/orientation for single-chain RWM is hard, or when benchmarking MCMC efficiency against WinBUGS or optimal-normal-jump random walk Metropolis.

## When NOT to use

- Discrete or non-real-parameter spaces — the difference-vector jump construction is defined over real parameter spaces only
- Posteriors with multiple distant modes when the initial population covers just a single modal region — far-away modes cannot be reached
- Problems needing a greedily fast sampler — DE-MC is explicitly not a greedy algorithm, a disadvantage in speed of convergence

## Inputs

- Target log-density / posterior definition over real parameter space (e.g. multivariate Normal or Student t3 with unit variances and pairwise correlations 0.5; one-way random-effects model with priors alpha=1, beta=10, a=1.85, b=0.1; 43-parameter Theophylline nonlinear mixed-effects model)
- Population size N (e.g. 2d, 3d, 10d chains) and jump scale gamma (default 2.38/sqrt(2d))
- Overdispersed initial chain states spanning the parameter space (narrow and broad starts for R-hat monitoring)
- Reference percentiles from WinBUGS 1.4 or analytically known true values

## Outputs

- Posterior draws per parameter (e.g. log(xi), phi, l Ke, l Ka, l Cl, log(tau2_e/a/c), log(sigma2)) from pooled post-burn-in samples
- Percentile tables (P2.5, P50, P97.5) with comparisons to WinBUGS or true values (e.g. efficiency_table.csv, percentiles_comparison.csv, demc_theophylline_percentiles.csv)
- Efficiency ratios 100 x MSE_RWM / MSE_DE-MC (reported 68%–501% across Normal and Student t3 targets)
- Convergence traces: Metropolis acceptance fraction (~0.23) and Gelman-Rubin R-hat per generation

## How to apply

Run N parallel chains initialized from overdispersed states and propose for each chain a jump equal to γ times the difference vector of two randomly chosen chains among the remaining N-1 chains, accepting via the Metropolis ratio (each jump is as likely as the reverse jump, making the N-chain system one random walk on an N×d space). Use the default γ = 2.38/sqrt(2d), decreasing γ somewhat if blocks are strongly correlated or to reach the optimal Metropolis acceptance rate (~0.23). Choose N relative to dimension — e.g. N = 2d, 3d, 10d — since efficiency relative to RWM rises as N/d grows, and monitor convergence with Gelman-Rubin R-hat across the overdispersed chains rather than independent chains. For high-dimensional posteriors such as the 43-parameter Theophylline nonlinear mixed-effects model, apply the block variant: partition the parameter vector into blocks (e.g. 12 per-patient blocks plus l Ke/l Ka/l Cl, tau, and sigma2 blocks), scale each block jump by γ = 2.38/sqrt(2*d_b), optionally apply binomial crossover, and run two inner block-update iterations per outer generation. Validate by comparing posterior percentiles (P2.5, P50, P97.5) against long-run WinBUGS references or true percentiles, expecting DE-MC MSE up to a factor of 4 lower for location parameters but up to a factor 2 higher for variance parameters.

## Related tools

- **demc2** (R package implementing DE-MC (and snooker updater variant) for adaptive MCMC on real parameter spaces; installed from GitHub) — CajoterBraak/demc2
- **remotes** (R package used to install demc2 from GitHub via remotes::install_github)
- **R** (Runtime environment in which DE-MC runs, target densities are defined, and percentile/efficiency tables are computed)
- **WinBUGS** (Reference sampler providing very-long-run posterior percentiles against which DE-MC results are compared)
- **TraitEnvMLMWA** (R package referenced alongside demc2 in the installation instructions of the associated repository)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Metropolis acceptance fraction of the N chains converges to approximately 0.23
- Gelman-Rubin R-hat across overdispersed chains drops below 1.2 (e.g. around 900–1000 generations on the d=100 Normal target)
- Efficiency ratios 100 x MSE_RWM / MSE_DE-MC fall within the reported 68%–501% range and increase as N/d grows
- DE-MC percentiles (P2.5/P50/P97.5) agree with WinBUGS reference or true percentiles within Monte Carlo error (at most ~0.02–0.1 discrepancies, which vanish with a tenfold longer burn-in)
- All chains initialized from overdispersed states mix to the same target distribution

## Limitations

- No theoretical analysis of the rate of convergence of DE-MC exists yet; it is much desired
- If the initial population covers just a single modal region, there is no chance that other modes far away can be reached
- DE-MC is not a greedy algorithm, a disadvantage in terms of speed of convergence; finding greedier variants that retain robustness and simplicity is an open challenge
- Large populations require more computer time to converge than small ones
- gamma may need to be decreased if blocks are strongly correlated, or somewhat in nonlinear mixed-effects models to reach the optimal acceptance rate
- Outside the generalized linear model context WinBUGS is limited to one-dimensional updates

## Evidence
