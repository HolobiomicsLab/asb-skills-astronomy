---
name: winbugs-reference-result-comparison
description: Use when a new MCMC sampler (e.g. DE-MC) must be validated against a
  trusted reference such as WinBUGS output or known analytical posterior percentiles
  for a Bayesian model.
when_to_use_negative:
- When the target model is a generalized linear model, where WinBUGS itself is adequate
  and its one-dimensional updates are not a limitation.
- When no analytically known true values or trusted reference sampler output exist
  for the model, since the comparison has no ground truth to validate against.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment for running the demc2 DE-MC sampler, computing percentiles, and
    building the comparison table
- name: demc2
  role: R package providing the Differential Evolution Markov Chain sampler used to
    generate the posterior draws being validated
  repo: https://github.com/CajoterBraak/demc2
- name: remotes
  role: Used to install demc2 from GitHub via remotes::install_github
- name: WinBUGS
  role: Reference MCMC software whose percentile output serves as the trusted comparison
    column for validation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:f0489cd71290a9ec789d6e399efb319483c3eefdf94c0bb1a45ce7127f634066
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# winbugs-reference-result-comparison

## Summary

Validates a DE-MC (Differential Evolution Markov Chain) implementation by tabulating its posterior percentiles against analytically known true values and a WinBUGS 1.4 reference column, as in the Liu & Hodges peak-discharge one-way random-effects example (Table 2).

## When to use

Use when a new MCMC sampler (e.g. DE-MC) must be validated against a trusted reference such as WinBUGS output or known analytical posterior percentiles for a Bayesian model.

## When NOT to use

- When the target model is a generalized linear model, where WinBUGS itself is adequate and its one-dimensional updates are not a limitation.
- When no analytically known true values or trusted reference sampler output exist for the model, since the comparison has no ground truth to validate against.

## Inputs

- Benchmark model specification (Liu & Hodges peak-discharge one-way random-effects model: I = 6, J = 4, priors alpha = 1, beta = 10, a = 1.85, b = 0.1)
- Posterior draws of monitored parameters (log(xi), phi) from DE-MC runs with population sizes N = 14, 21, 70
- Analytically known true posterior percentiles of log(xi) and phi
- WinBUGS 1.4 reference percentile column for the same model

## Outputs

- Comparison table of DE-MC percentiles vs true values vs WinBUGS 1.4 percentiles for log(xi) and phi at each N (e.g. percentiles_comparison.csv)
- Maximum absolute percentile deviation per parameter and N, and burn-in sensitivity assessment for residual discrepancies

## How to apply

Set up the benchmark model exactly as in the paper: the Liu & Hodges peak-discharge one-way random-effects model with I = 6 groups, J = 4 observations per group, and priors alpha = 1, beta = 10, a = 1.85, b = 0.1. Run DE-MC from an overdispersed initial population of N chains (e.g. N = 14, 21, 70) using the jump scale gamma = 2.38/sqrt(2d), with gamma = 1 every 10th generation, and extract posterior draws of log(xi) and phi. Compute the percentiles of log(xi) and phi for each N and tabulate them side-by-side with the analytically known true values and the WinBUGS 1.4 reference column. Judge success by percentile agreement: with N = 14 the DE-MC percentiles differed from the truth by at most 0.02; residual discrepancies (e.g. 0.1 at the 97.5% point of log(xi) for N = 21 and 70) should be checked by extending the burn-in tenfold, which removed the discrepancy. Save the comparison as a table (e.g. percentiles_comparison.csv).

## Related tools

- **R** (Environment for running the demc2 DE-MC sampler, computing percentiles, and building the comparison table)
- **demc2** (R package providing the Differential Evolution Markov Chain sampler used to generate the posterior draws being validated) — https://github.com/CajoterBraak/demc2
- **remotes** (Used to install demc2 from GitHub via remotes::install_github)
- **WinBUGS** (Reference MCMC software whose percentile output serves as the trusted comparison column for validation)

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- With N = 14, DE-MC percentiles of log(xi) and phi differ from the true percentiles by at most 0.02.
- Any 0.1-scale discrepancy at the 97.5% point of log(xi) for N = 21 or 70 disappears after a tenfold longer burn-in.
- DE-MC percentile columns match the WinBUGS 1.4 reference column within the same tolerance.
- The jump scale used matches gamma = 2.38/sqrt(2d) with gamma = 1 applied every 10th generation.
- Comparison table is saved as percentiles_comparison.csv with one row per percentile/parameter/N combination.

## Limitations

- Large populations require more computer time to converge than small ones.
- If the initial population covers just a single modal region, distant modes cannot be reached, so overdispersed initialization matters for a fair comparison.
- If blocks are strongly correlated, gamma = 2.38/sqrt(2d) may need to be decreased; in the nonlinear mixed-effects model gamma had to be decreased somewhat for an optimal acceptance rate.
- No theoretical analysis of the rate of convergence of DE-MC exists yet, so convergence must be judged empirically (e.g. by burn-in sensitivity).

## Evidence

- [results] DE-MC results with N = 14 differed at most 0.02 from the true percentiles of log(xi) and phi: [evidence span withheld — anchor status: not_found]
- [results] 0.1 discrepancy disappearing with longer burn-in: [evidence span withheld — anchor status: not_found]
