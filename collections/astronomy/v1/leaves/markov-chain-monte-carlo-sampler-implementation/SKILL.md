---
name: markov-chain-monte-carlo-sampler-implementation
description: Use when sampling from a Bayesian posterior or other target density on
  a real parameter space of dimension d where choosing the scale and orientation of
  a jumping distribution is hard, and block/multi-chain updating is desired.
when_to_use_negative:
- The initial population covers only a single modal region — distant modes cannot
  be reached, so DE-MC is wrong for far-away mode discovery in that setting
- Speed of convergence is critical and a greedy optimizer is acceptable — DE-MC is
  not a greedy algorithm, which the paper notes is a disadvantage
- Large populations are required — the paper notes large populations require more
  computer time to converge than small ones
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Implementation language for the ~10-line core DE-MC sampler loop and the exported
    de_mc_sampler.R module
- name: remotes
  role: Install helper used to fetch the demc2 R library providing Differential Evolution
    Markov Chain parameter estimation
- name: demc2
  role: R library for Adaptive MCMC on real parameter spaces via Differential Evolution
    Markov Chain, implementing this skill
  repo: https://github.com/CajoterBraak/demc2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:2c50cf6fbde95f75d84d7077192d4e3842f087cbfecfb55f81d0e1effd7ee628
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - demc2
  - remotes
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# markov-chain-monte-carlo-sampler-implementation

## Summary

Implement the DE-MC sampler of ter Braak (2006): a multi-chain Metropolis MCMC whose jump is x_p = x_i + γ(x_R1 − x_R2) + e, where the difference of two randomly chosen other chains supplies scale and orientation adapted to the target's variance-covariance matrix.

## When to use

Use when sampling from a Bayesian posterior or other target density on a real parameter space of dimension d where choosing the scale and orientation of a jumping distribution is hard, and block/multi-chain updating is desired.

## When NOT to use

- The initial population covers only a single modal region — distant modes cannot be reached, so DE-MC is wrong for far-away mode discovery in that setting
- Speed of convergence is critical and a greedy optimizer is acceptable — DE-MC is not a greedy algorithm, which the paper notes is a disadvantage
- Large populations are required — the paper notes large populations require more computer time to converge than small ones

## Inputs

- Target density pi(x) defined on R^d (e.g. unnormalized posterior from a Bayesian model)
- Population size N (number of parallel chains, N > 2)
- Overdispersed initial states for the N chains in R^d
- Scaling factor gamma (default 2.38/sqrt(2d)) and perturbation variance b for e ~ N(0, b·I)

## Outputs

- R function de_mc_sampler.R (~10 lines) implementing the proposal-and-Metropolis loop
- Posterior samples: N parallel chains on R^d after burn-in
- Acceptance-rate diagnostics used to tune gamma

## How to apply

Initialize N chains from overdispersed starting states in the d-dimensional target space. For each chain i, draw two chains R1 and R2 at random from the remaining N−1 chains and form the proposal x_p = x_i + γ(x_R1 − x_R2) + e, using the default scaling γ = 2.38/sqrt(2d); e is drawn from a symmetric distribution with small variance but unbounded support (e.g. e ~ N(0, b·I), b small) so the whole parameter space can be reached. Accept with probability min(1, pi(x_p)/pi(x_i)); because each jump is as likely as the reverse jump given the current state of the remaining chains, this is a valid Metropolis step on the joint N×d space. Decrease γ if blocks are strongly correlated or to reach an optimal acceptance rate (as needed for the nonlinear mixed-effects model). Wrap the core loop in ~10 lines of R and export it as de_mc_sampler.R.

## Related tools

- **R** (Implementation language for the ~10-line core DE-MC sampler loop and the exported de_mc_sampler.R module)
- **remotes** (Install helper used to fetch the demc2 R library providing Differential Evolution Markov Chain parameter estimation)
- **demc2** (R library for Adaptive MCMC on real parameter spaces via Differential Evolution Markov Chain, implementing this skill) — https://github.com/CajoterBraak/demc2

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Each proposal satisfies the symmetric-jump Metropolis criterion: each jump is as likely as the reverse jump given the current state of the remaining chains
- The module runs without error on a simple test target (e.g. normal or Student distributions used in the paper's demonstrations)
- The scaling factor equals the default γ = 2.38/sqrt(2d) unless a decrease is justified (e.g. strongly correlated blocks or acceptance-rate tuning)
- Acceptance rate is in an optimal range; γ needed to be decreased somewhat in the paper's nonlinear mixed-effects example
- Chains initialized from overdispersed states converge to the target's variance-covariance structure, since jump scale and orientation adapt automatically

## Limitations

- No theoretical analysis of the rate of convergence of DE-MC exists yet; the authors state it is much desired
- If the initial population covers just a single modal region, distant modes cannot be reached
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence
- Large populations require more computer time to converge than small ones
- γ may need to be decreased if blocks are strongly correlated or to get an optimal acceptance rate

## Evidence

- [methods] The DE-MC proposal is x_p = x_i + γ(x_R1 − x_R2) + e, where e is drawn from a symmetric distribution with a small variance: [evidence span withheld — anchor status: not_found]
