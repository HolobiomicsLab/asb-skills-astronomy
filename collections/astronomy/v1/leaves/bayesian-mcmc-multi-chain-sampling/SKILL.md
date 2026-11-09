---
name: bayesian-mcmc-multi-chain-sampling
description: Use when a Bayesian model over real parameter spaces needs MCMC sampling
  but choosing the scale/orientation of the jumping distribution is difficult, or
  when comparing DE-MC against WinBUGS-style single-site samplers on posterior percentile
  accuracy.
when_to_use_negative:
- When the initial population covers just a single modal region and distant modes
  must be reached — DE-MC cannot reach far-away modes from a single modal region
- When a closed-form Gibbs sampler is available for the target (e.g. conditional distributions
  easy to sample directly), a standard Gibbs sampler may be preferable
- Outside the generalized linear model context where WinBUGS one-dimensional updates
  already suffice and DE-MC's block updating offers no advantage
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment in which the demc2 package is installed and DE-MC / Block DE-MC
    sampling and RMSE computation are run
- name: remotes
  role: Used to install the demc2 package from GitHub via remotes::install_github
- name: WinBUGS
  role: Reference sampler providing one-dimensional-update MCMC results for RMSE and
    timing comparison against DE-MC
- name: demc2 (CajoterBraak/demc2)
  role: R library providing Differential Evolution Markov Chain parameter estimation
    / Adaptive MCMC on real parameter spaces
  repo: https://github.com/CajoterBraak/demc2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:8aa61cf784eecac4be126aae40a008aae7c14db47f7d74666056d142ec611166
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - WinBUGS
  - demc2 (CajoterBraak/demc2)
  - remotes
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# bayesian-mcmc-multi-chain-sampling

## Summary

Run the Differential Evolution Markov Chain (DE-MC) sampler on a Bayesian target model in R using the demc2 package, with multiple overdispersed chains that learn from each other, and benchmark its accuracy against WinBUGS one-dimensional updates via RMSE of posterior quantiles.

## When to use

Use when a Bayesian model over real parameter spaces needs MCMC sampling but choosing the scale/orientation of the jumping distribution is difficult, or when comparing DE-MC against WinBUGS-style single-site samplers on posterior percentile accuracy.

## When NOT to use

- When the initial population covers just a single modal region and distant modes must be reached — DE-MC cannot reach far-away modes from a single modal region
- When a closed-form Gibbs sampler is available for the target (e.g. conditional distributions easy to sample directly), a standard Gibbs sampler may be preferable
- Outside the generalized linear model context where WinBUGS one-dimensional updates already suffice and DE-MC's block updating offers no advantage

## Inputs

- Bayesian target model specification over real parameter spaces (e.g. seven-parameter Theophylline nonlinear mixed-effects model)
- Reference posterior values/percentiles for validation
- R session with remotes package
- WinBUGS installation for the reference one-dimensional-update sampler

## Outputs

- Posterior samples from N parallel DE-MC chains for each parameter
- Convergence diagnostics (maximum R-hat across parameters)
- RMSE of P2.5, P50, P97.5 posterior quantile estimates per parameter per sampler
- rmse_table.csv with RMSE and minutes-per-simulation runtimes for WinBUGS, DE-MC (N=86) and Block DE-MC (N=9)

## How to apply

Install demc2 from GitHub with remotes::install_github("CajoterBraak/demc2") after install.packages("remotes"). Set up the target Bayesian model (e.g. the seven-parameter Theophylline nonlinear mixed-effects model) together with reference posterior values. Run DE-MC with multiple chains initialized from overdispersed states, using the default jump scale gamma = 2.38/sqrt(2d) so jumps derive from differences of two randomly chosen chains scaled by gamma; for Block DE-MC, decrease gamma if blocks are strongly correlated to reach an optimal acceptance rate (as needed in the nonlinear mixed-effects model). Optionally run WinBUGS on the same model as the one-dimensional-update reference sampler. Discard 20% burn-in, verify convergence by requiring the maximum R-hat across parameters to be below 1.2, then compute RMSE of the P2.5, P50, and P97.5 posterior quantile estimates per parameter.

## Related tools

- **R** (Environment in which the demc2 package is installed and DE-MC / Block DE-MC sampling and RMSE computation are run)
- **remotes** (Used to install the demc2 package from GitHub via remotes::install_github)
- **WinBUGS** (Reference sampler providing one-dimensional-update MCMC results for RMSE and timing comparison against DE-MC)
- **demc2 (CajoterBraak/demc2)** (R library providing Differential Evolution Markov Chain parameter estimation / Adaptive MCMC on real parameter spaces) — https://github.com/CajoterBraak/demc2

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Maximum R-hat across all parameters is below 1.2 after discarding the 20% burn-in from each chain
- RMSE of DE-MC posterior percentile estimates (P2.5, P50, P97.5) for location parameters (l Ke, l Ka, l Cl) is up to a factor of 4 lower than WinBUGS
- Default jump scale gamma = 2.38/sqrt(2d) yields good acceptance rates; acceptance rate is optimized by decreasing gamma only when blocks are strongly correlated
- rmse_table.csv is produced containing RMSE values for seven parameters and minutes-per-simulation runtimes for each sampler (WinBUGS, DE-MC N=86, Block DE-MC N=9)
- Chains are initialized from overdispersed states and run in parallel as a single random walk Markov chain on the N×d-dimensional space

## Limitations

- No theoretical analysis of the rate of convergence of DE-MC exists yet; the authors state it is much desired
- If the initial population covers just a single modal region, distant modes cannot be reached
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence; large populations require more computer time to converge than small ones
- In the nonlinear mixed-effects model, gamma needed to be decreased somewhat to get an optimal acceptance rate
- The conditional distribution required for Gibbs sampling gamma is often not available in closed form or not easy to sample from directly

## Evidence

- [intro] we run multiple Markov chains, which are initialized from overdispersed states, in parallel and let the chains learn from each other: "we run multiple Markov chains, which are initialized from overdispersed states, in parallel and let the chains learn from each other"
- [methods] remotes::install_github("CajoterBraak/demc2"): "remotes::install_github("CajoterBraak/demc2")"
- [discussion] The suggested default value of γ = 2.38 / √ (2d) performed well in bock updating (Table 4): "The suggested default value of γ = 2 . 38 / √ (2d) performed well in bock updating (Table 4)"
- [readme] Differential Evolution Markov Chain parameter estimation R library for Adaptive MCMC on real parameter spaces: "Differential Evolution Markov Chain parameter estimation R library for Adaptive MCMC on real parameter spaces via Differential Evolution Markov Chain"
- [discussion] In the nonlinear mixed-effects model, γ needed to be decreased somewhat to get an optimal accep- tance rate.: "In the nonlinear mixed-effects model, γ needed to be decreased somewhat to get an optimal accep- tance rate."
