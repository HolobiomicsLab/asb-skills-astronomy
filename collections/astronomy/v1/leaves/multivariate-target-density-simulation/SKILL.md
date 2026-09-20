---
name: multivariate-target-density-simulation
description: Use when benchmarking a multi-chain MCMC sampler (e.g. DE-MC) against
  a reference sampler on target distributions of known density, so that percentile-estimate
  MSE and sampling efficiency can be computed exactly.
when_to_use_negative:
- The target distribution is unknown (a real Bayesian posterior) — efficiency vs optimal-normal-jump
  RWM cannot be validated against known percentile values
- The parameter space is not real-valued continuous (e.g. discrete or one-dimensional
  updates) — outside the DE-MC real parameter space scope, where WinBUGS-style one-dimensional
  updates apply
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Language used to define the multivariate Normal and Student t3 target densities
    and run the DE-MC and RWM samplers
- name: remotes
  role: Package installer used to fetch demc2 from GitHub
- name: demc2
  role: R library providing Differential Evolution Markov Chain adaptive MCMC on real
    parameter spaces used to sample the simulated targets
  repo: https://github.com/CajoterBraak/demc2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:5a462df8acebcb0bc47188ffb04f37caca94799cb82f907af69da12ae550c91f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - TraitEnvMLMWA
  - demc2
  - remotes
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# multivariate-target-density-simulation

## Summary

Constructs multivariate Normal and Student t3 target densities with unit variances and pairwise correlations of 0.5 (d = 5, 50, 100) as known-shape benchmark distributions for validating MCMC samplers such as DE-MC against single-chain random walk Metropolis.

## When to use

Use when benchmarking a multi-chain MCMC sampler (e.g. DE-MC) against a reference sampler on target distributions of known density, so that percentile-estimate MSE and sampling efficiency can be computed exactly.

## When NOT to use

- The target distribution is unknown (a real Bayesian posterior) — efficiency vs optimal-normal-jump RWM cannot be validated against known percentile values
- The parameter space is not real-valued continuous (e.g. discrete or one-dimensional updates) — outside the DE-MC real parameter space scope, where WinBUGS-style one-dimensional updates apply

## Inputs

- Target specification: multivariate Normal or multivariate Student t3 density with unit variances (variance j) and pairwise correlations of 0.5
- Dimension d in {5, 50, 100}
- DE-MC population size N in {2d, 3d, 10d} with overdispersed initial chain states
- Multiple replicates per (target, d, N) configuration

## Outputs

- Posterior draws from DE-MC and from the single-chain RWM reference sampler for each configuration
- Mean squared error (MSE) of P50 and pooled P2.5 percentile estimates per sampler
- efficiency_table.csv with efficiency ratios 100 x MSE_RWM / MSE_DE-MC (reported range 68%-501%)

## How to apply

Define multivariate target densities in R with unit variances (variance j) and pairwise correlations of 0.5 for each dimension d = 5, 50, 100, using both multivariate Normal and multivariate Student t3 shapes. Initialize DE-MC from overdispersed chains with population size N = 2d, 3d, 10d, using the default jump scale gamma = 2.38/sqrt(2d) applied to the difference of two randomly chosen chains among the remaining N-1 chains. Run each (target, d, N) configuration in parallel with a single-chain random walk Metropolis reference sampler, collecting posterior draws for P50 and pooled P2.5/P97.5 percentile estimates across replicates. Compute MSE per percentile estimate for both samplers and tabulate efficiency ratios 100 x MSE_RWM / MSE_DE-MC; correct application reproduces ratios in the reported 68%-501% range, rising as N/d grows.

## Related tools

- **R** (Language used to define the multivariate Normal and Student t3 target densities and run the DE-MC and RWM samplers)
- **remotes** (Package installer used to fetch demc2 from GitHub)
- **demc2** (R library providing Differential Evolution Markov Chain adaptive MCMC on real parameter spaces used to sample the simulated targets) — https://github.com/CajoterBraak/demc2

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Computed efficiency ratios 100 x MSE_RWM / MSE_DE-MC fall within the reported 68%-501% range across the (d, N) grid for both Normal and Student t3 targets
- Efficiency increases monotonically as N/d grows (2d < 3d < 10d) for a given d
- Pooled 2.5/97.5 percentile estimates show higher efficiency than median (P50) estimates
- Correlation structure of sampled draws matches the specified 0.5 pairwise correlations and unit variances of the target
- t3 targets yield lower acceptance-rate/efficiency behavior than Normal targets due to heavier tails

## Limitations

- If the initial population covers just a single modal region, distant modes cannot be reached — overdispersed initialization matters
- Large populations (e.g. N = 10d) require more computer time to converge than small ones
- gamma may need to be decreased if blocks are strongly correlated
- No theoretical analysis of the rate of convergence of DE-MC exists

## Evidence

- [methods] Define the multivariate Normal and Student t3 target densities with unit variances (variance j) and pairwise correlations of 0.5 for d = 5, 50, 100.: [evidence span withheld — anchor status: not_found]
- [methods] Tabulate efficiency ratios: [evidence span withheld — anchor status: not_found]
