---
name: markov-chain-monte-carlo-target-density-specification
description: Use when you need to draw samples from a real-parameter-space target
  density (e.g. a high-dimensional posterior) and want an MCMC that avoids hand-tuning
  the jump distribution's scale and orientation.
when_to_use_negative:
- If the initial population covers just a single modal region, distant modes cannot
  be reached — do not expect DE-MC to discover far-away modes from a single-modal
  start.
- When you need one-dimensional block updates inside a generalized linear model context,
  WinBUGS-style samplers may suffice; DE-MC targets real parameter spaces where jump
  scale/orientation choice is the bottleneck.
- 'When a greedy optimizer is needed for speed: DE-MC is not a greedy algorithm, which
  is a disadvantage in terms of speed of convergence.'
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: demc2
  role: R library implementing Differential Evolution Markov Chain for adaptive MCMC
    on real parameter spaces; installed from GitHub to run the N-chain sampling and
    convergence monitoring
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: R package used to install demc2 from GitHub via remotes::install_github
- name: R
  role: Environment in which the demc2 package is installed and the target density,
    populations, and convergence traces are computed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:95fab6ef21369950c29bbba726efb6e1c4156f17d2a2abd86c02b732af5098fe
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - TraitEnvMLMWA
  - demc2
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# markov-chain-monte-carlo-target-density-specification

## Summary

Specify a target log-density (e.g. a d=100 standard Normal) and sample it with Differential Evolution Markov Chain (DE-MC), in which N parallel chains initialized from overdispersed states learn from each other and self-tune jump scale and orientation to the target's variance-covariance matrix.

## When to use

Use when you need to draw samples from a real-parameter-space target density (e.g. a high-dimensional posterior) and want an MCMC that avoids hand-tuning the jump distribution's scale and orientation.

## When NOT to use

- If the initial population covers just a single modal region, distant modes cannot be reached — do not expect DE-MC to discover far-away modes from a single-modal start.
- When you need one-dimensional block updates inside a generalized linear model context, WinBUGS-style samplers may suffice; DE-MC targets real parameter spaces where jump scale/orientation choice is the bottleneck.
- When a greedy optimizer is needed for speed: DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence.

## Inputs

- Target log-density function over a real parameter space (e.g. d=100 standard Normal)
- Initial population of N chains in real parameter space, provided as narrow and overdispersed starting states
- Population size N (e.g. 101, 200, 1000) and jump scaling gamma (default 2.38/sqrt(2d))

## Outputs

- Posterior samples for each of the N chains across generations
- Per-generation Metropolis acceptance fraction trace of the N chains
- Per-generation Gelman-Rubin R-hat trace across overdispersed chains
- Generations-to-convergence summary per N (e.g. demc_d100_normal_results.csv)

## How to apply

Define the target's log-density explicitly (e.g. the d=100 standard Normal used in the paper) and generate two initial populations per population size N — one narrow, one broad/overdispersed — so the parallel chains can be used for convergence checking. Run DE-MC with N chains whose jump is the difference vector of two randomly chosen chains among the remaining N-1 chains, multiplied by gamma and added to the current chain's vector; use the default gamma = 2.38/sqrt(2d), which the paper found to perform well, decreasing it only if blocks are strongly correlated. Each proposal defines a Metropolis step on the N x d-dimensional space, making the N chains a single random walk Markov chain. Monitor the Metropolis acceptance fraction per generation, expecting it to converge to approximately 0.23, and compute the Gelman-Rubin R-hat across the overdispersed chains, recording the generation at which R-hat first drops below 1.2. Tabulate generations-to-convergence per N (for the d=100 Normal: about 4000, 1000 and 600 generations for N = 101, 200, 1000) and export acceptance/R-hat traces plus a convergence summary (e.g. to a CSV file).

## Related tools

- **demc2** (R library implementing Differential Evolution Markov Chain for adaptive MCMC on real parameter spaces; installed from GitHub to run the N-chain sampling and convergence monitoring) — https://github.com/CajoterBraak/demc2
- **remotes** (R package used to install demc2 from GitHub via remotes::install_github)
- **R** (Environment in which the demc2 package is installed and the target density, populations, and convergence traces are computed)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Metropolis acceptance fraction converges to approximately 0.23 across the N chains
- Gelman-Rubin R-hat across the overdispersed chains first drops below 1.2 at a recorded generation (expected around 900-1000 generations for the d=100 Normal case)
- Generations-to-convergence matches the reported pattern for the d=100 Normal target: about 4000, 1000 and 600 generations for N = 101, 200 and 1000
- Function-evaluation totals follow the reported trade-off: 404,000, 200,000 and 600,000 evaluations for N = 101, 200, 1000, fastest in draws for N = 200
- Output CSV (e.g. demc_d100_normal_results.csv) contains acceptance traces, R-hat traces and the per-N convergence summary

## Limitations

- No theoretical analysis of the rate of convergence of DE-MC exists yet; the authors state it is much desired.
- If the initial population covers just a single modal region, distant modes cannot be reached.
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence; large populations require more computer time to converge than small ones.
- gamma may need to be decreased if blocks are strongly correlated (e.g. in the nonlinear mixed-effects model, gamma needed to be decreased somewhat to get an optimal acceptance rate).

## Evidence
