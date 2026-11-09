---
name: metropolis-acceptance-criterion-verification
description: Use when implementing or auditing an MCMC sampler with difference-vector
  proposals (DE-MC scheme DE1) and you must confirm the proposal kernel is symmetric
  so a plain Metropolis ratio pi(x_p)/pi(x_i) is valid.
when_to_use_negative:
- When the proposal kernel is not symmetric (e.g. asymmetric or independent proposals),
  where a full Hastings ratio q(x_i, x_p)/q(x_p, x_i) is required instead of the plain
  Metropolis ratio.
- When the initial population covers only a single modal region and distant modes
  must be reached — the sampler cannot jump to far-away modes, so verification of
  the acceptance criterion will not fix the coverage problem.
- When one-dimensional updates suffice inside a generalized linear model context,
  where WinBUGS-style samplers are already adequate.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Language used to implement the DE-MC sampler and its Metropolis acceptance
    step; the core method can be coded in about 10 lines.
- name: demc2
  role: R library providing Differential Evolution Markov Chain parameter estimation
    / Adaptive MCMC on real parameter spaces, installable via remotes::install_github.
  repo: https://github.com/CajoterBraak/demc2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11222-006-8769-1
    title: 'A Markov Chain Monte Carlo version of the genetic algorithm Differential
      Evolution: easy Bayesian computing for real parameter spaces'
  source_content_hash: sha256:75863bb32caebd7900eb7773591a6fa340af3c4f80529e1ea15a554b1bbb03cb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CajoterBraak/demc2
  tools:
  - R
  - demc2
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metropolis-acceptance-criterion-verification

## Summary

Verifies that a Differential Evolution Markov Chain (DE-MC) proposal defines a valid Metropolis step: because x_p = x_i + gamma*(x_R1 - x_R2) + e with symmetric random perturbation e, each jump must be as likely as the reverse jump given the current state of the remaining chains, and acceptance must follow min(1, pi(x_p)/pi(x_i)).

## When to use

Use when implementing or auditing an MCMC sampler with difference-vector proposals (DE-MC scheme DE1) and you must confirm the proposal kernel is symmetric so a plain Metropolis ratio pi(x_p)/pi(x_i) is valid.

## When NOT to use

- When the proposal kernel is not symmetric (e.g. asymmetric or independent proposals), where a full Hastings ratio q(x_i, x_p)/q(x_p, x_i) is required instead of the plain Metropolis ratio.
- When the initial population covers only a single modal region and distant modes must be reached — the sampler cannot jump to far-away modes, so verification of the acceptance criterion will not fix the coverage problem.
- When one-dimensional updates suffice inside a generalized linear model context, where WinBUGS-style samplers are already adequate.

## Inputs

- Current population of N chain states (N x d matrix) in the target real parameter space
- Unnormalized target density pi(x) (e.g. a Bayesian posterior)
- Scaling factor gamma (default 2.38/sqrt(2d))
- Symmetric perturbation distribution e ~ N(0, b*I) with small b and unbounded support

## Outputs

- Accept/reject decision per proposal with acceptance probability min(1, pi(x_p)/pi(x_i))
- Verified symmetric-jump property for the proposal kernel on the joint N x d space
- Empirical acceptance rate used to judge whether gamma needs adjustment

## How to apply

Initialize N chains from overdispersed states in the d-dimensional target space; for each chain i draw two chains R1 and R2 at random from the remaining N-1 chains and compute the proposal x_p = x_i + gamma*(x_R1 - x_R2) + e, with default scaling gamma = 2.38/sqrt(2d). The perturbation e must be drawn from a symmetric distribution with small variance relative to the target but unbounded support (e.g. e ~ N(0, b*I) with b small) so the whole parameter space can be reached and each jump is as likely as its reverse given the fixed states of the remaining chains. Accept with probability min(1, pi(x_p)/pi(x_i)); this Metropolis step makes the N-chain system a single random walk Markov chain on the N x d-dimensional space. Rationale: the difference vector (x_R1 - x_R2) carries scale and orientation information of the target, so it automatically adapts to the variance-covariance matrix, but this adaptive property only preserves detailed balance if the symmetry condition holds — which is exactly what this verification checks.

## Related tools

- **R** (Language used to implement the DE-MC sampler and its Metropolis acceptance step; the core method can be coded in about 10 lines.)
- **demc2** (R library providing Differential Evolution Markov Chain parameter estimation / Adaptive MCMC on real parameter spaces, installable via remotes::install_github.) — https://github.com/CajoterBraak/demc2

## Examples

```
install.packages("remotes"); remotes::install_github("CajoterBraak/demc2")
```

## Evaluation signals

- Each proposal satisfies the symmetric-jump Metropolis criterion: each jump is as likely as the reverse jump given the current state of the remaining chains.
- Acceptance decision equals min(1, pi(x_p)/pi(x_i)) for the unnormalized target density pi.
- The module (e.g. de_mc_sampler.R, ~10 lines for the core) runs without error on a simple test target such as a normal or Student distribution.
- Empirical acceptance rate is near optimal; if gamma = 2.38/sqrt(2d) underperforms (e.g. with strongly correlated blocks), gamma may need to be decreased.
- The perturbation e has unbounded support so the whole parameter space can be reached — no hard boundaries in the sampling of e.

## Limitations

- No theoretical analysis of the rate of convergence of DE-MC exists yet; the authors state it is much desired.
- If the initial population covers just a single modal region, distant modes cannot be reached.
- gamma may need to be decreased if blocks are strongly correlated or, as in the nonlinear mixed-effects model, to obtain an optimal acceptance rate.
- DE-MC is not a greedy algorithm, which is a disadvantage in terms of speed of convergence; large populations require more computer time to converge than small ones.

## Evidence
