---
name: metropolis-acceptance-rate-monitoring
description: Use when running DE-MC (or another Metropolis-based multi-chain sampler)
  and you need to verify tuning of the jump scale γ via the per-generation acceptance
  fraction, e.g. checking it converges to ~0.23 on a high-dimensional target.
when_to_use_negative:
- The task is a one-dimensional Gibbs update in a generalized linear model context
  — the article notes WinBUGS suffices there and DE-MC's N×d population machinery
  is unnecessary
- The initial population covers only a single modal region and distant modes must
  be reached — acceptance-rate monitoring cannot fix this, as far-away modes are unreachable
- Blocks are strongly correlated and γ has not been decreased — the acceptance fraction
  will be off-optimal and the ~0.23 benchmark no longer applies until γ is retuned
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: demc2
  role: R package implementing Differential Evolution Markov Chain; run per generation
    to obtain the Metropolis acceptance fraction of the N chains
  repo: CajoterBraak/demc2
- name: remotes
  role: Used to install demc2 from GitHub via remotes::install_github
- name: R
  role: Runtime environment in which the target log-density, DE-MC runs, acceptance
    and R-hat traces are computed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:e064eb48452d9489bae3e664718d197359c3ab3d5b542bcd36a20b073d7d3d0b
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
yes_rate: 0.6428571428571429
n_claims: 14
---

# metropolis-acceptance-rate-monitoring

## Summary

Monitor the Metropolis acceptance fraction across the parallel chains of a DE-MC run (demc2) to verify that the jump scale γ = 2.38/sqrt(2d) yields an optimal acceptance rate (~0.23), alongside Gelman-Rubin R-hat convergence checks on the d=100 Normal target.

## When to use

Use when running DE-MC (or another Metropolis-based multi-chain sampler) and you need to verify tuning of the jump scale γ via the per-generation acceptance fraction, e.g. checking it converges to ~0.23 on a high-dimensional target.

## When NOT to use

- The task is a one-dimensional Gibbs update in a generalized linear model context — the article notes WinBUGS suffices there and DE-MC's N×d population machinery is unnecessary
- The initial population covers only a single modal region and distant modes must be reached — acceptance-rate monitoring cannot fix this, as far-away modes are unreachable
- Blocks are strongly correlated and γ has not been decreased — the acceptance fraction will be off-optimal and the ~0.23 benchmark no longer applies until γ is retuned

## Inputs

- Target log-density function (e.g. d=100 standard Normal log-density) defined in R
- Two initial populations per population size N (narrow and broad/overdispersed starting states for the parallel chains)
- Population size N (e.g. 101, 200, 1000) and jump scaling γ (default 2.38/sqrt(2d))

## Outputs

- Per-generation Metropolis acceptance fraction trace across the N chains
- Per-generation Gelman-Rubin R-hat trace across overdispersed chains
- Convergence summary table of generations-to-convergence and function evaluations (e.g. demc_d100_normal_results.csv)

## How to apply

Run DE-MC via the demc2 R package (installed from GitHub with remotes::install_github) on the target log-density, using the default jump scaling γ = 2.38/sqrt(2d), which lets jump scale and orientation adapt to the target's variance-covariance matrix. At each generation record the Metropolis acceptance fraction of the N chains (e.g. N = 101, 200, 1000 for the d=100 standard Normal target initialized from narrow and overdispersed starting states). Check that the acceptance fraction converges to approximately 0.23; if the target blocks are strongly correlated or the model is nonlinear mixed-effects-like, decrease γ to recover the optimal acceptance rate. Simultaneously compute the Gelman-Rubin R-hat statistic across overdispersed chains per generation and record when it first drops below 1.2 (expected around 900–1000 generations for this target). Tabulate generations-to-convergence (about 4000, 1000 and 600 for N = 101, 200, 1000, i.e. 404,000 / 200,000 / 600,000 function evaluations) and write acceptance and R-hat traces to a results CSV.

## Related tools

- **demc2** (R package implementing Differential Evolution Markov Chain; run per generation to obtain the Metropolis acceptance fraction of the N chains) — CajoterBraak/demc2
- **remotes** (Used to install demc2 from GitHub via remotes::install_github)
- **R** (Runtime environment in which the target log-density, DE-MC runs, acceptance and R-hat traces are computed)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- The Metropolis acceptance fraction converges to approximately 0.23 at the default γ = 2.38/sqrt(2d)
- Gelman-Rubin R-hat across the overdispersed chains first drops below 1.2 around 900–1000 generations for the d=100 Normal target
- Generations-to-convergence match ~4000, 1000 and 600 for N = 101, 200 and 1000 (≈404,000 / 200,000 / 600,000 function evaluations)
- Acceptance and R-hat traces plus convergence summary are written to demc_d100_normal_results.csv without missing generations
- N = 200 is fastest in draws (200,000 function evaluations), consistent with the reported finding

## Limitations

- Large populations require more computer time to converge than small ones
- If blocks are strongly correlated, γ may need to be decreased; in the nonlinear mixed-effects model γ needed to be decreased somewhat to get an optimal acceptance rate
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence; no theoretical analysis of its rate of convergence exists yet
- If the initial population covers just a single modal region, distant modes cannot be reached

## Evidence

- [results] workflow acceptance monitoring: [evidence span withheld — anchor status: not_found]
