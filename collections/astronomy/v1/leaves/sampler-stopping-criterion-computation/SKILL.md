---
name: sampler-stopping-criterion-computation
description: 'Decision support for stopping or comparing MCMC runs: given autocorrelation
  times, walker counts, burn-in and sample lengths, determine whether the sample has
  reached a target error regime (e.g. the 10% level) versus a competing sampler''s
  tuning endpoint.'
when_to_use_negative:
- When only a single fixed chain with no burn-in/autocorrelation metadata exists and
  no error target is specified — epsilon is undefined without b, n, L and tau
- When the goal is proposal tuning of a Metropolis–Hastings sampler (the article's
  comparison endpoint n = 2000 is specific to the self-tuning CosmoMC benchmark, not
  general proposal adaptation)
edam_operation: http://edamontology.org/operation_0337
tools:
- name: Python
  role: Language in which the epsilon = |2*tau/((n-b)*L)|^(1/2) computation is evaluated
    with numpy and plotted with matplotlib
- name: acor
  role: Python wrapper estimating the autocorrelation time tau of MCMC time-series
    chains used in the epsilon formula
  repo: https://github.com/dfm/acor
- name: CosmoHammer
  role: Framework embedding the emcee affine-invariant ensemble sampler whose walker
    count L and step count n supply the (L, n) grid of the stopping analysis
  repo: https://github.com/cosmo-ethz/CosmoHammer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:33eb057df544788e35f66c42ee792f61463fd61a552f972a6b499583b4eb8ee1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cosmo-ethz/CosmoHammer
  repo_url: https://github.com/dfm/acor
  tools:
  - CosmoHammer
  - Python
  - acor
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sampler-stopping-criterion-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the statistical error epsilon of an MCMC ensemble sample from the autocorrelation time tau, burn-in length b, sequential steps n, and walker count L via epsilon = |2*tau/((n-b)*L)|^(1/2), to decide when sampling effort suffices (e.g. epsilon <= 0.1).

## When to use

Decision support for stopping or comparing MCMC runs: given autocorrelation times, walker counts, burn-in and sample lengths, determine whether the sample has reached a target error regime (e.g. the 10% level) versus a competing sampler's tuning endpoint.

## When NOT to use

- When only a single fixed chain with no burn-in/autocorrelation metadata exists and no error target is specified — epsilon is undefined without b, n, L and tau
- When the goal is proposal tuning of a Metropolis–Hastings sampler (the article's comparison endpoint n = 2000 is specific to the self-tuning CosmoMC benchmark, not general proposal adaptation)

## Inputs

- Autocorrelation times tau from Table 2 of arXiv:1212.1721, or raw MCMC chains passed to acor
- Grid of walker counts L and sequential step counts n from the benchmark configuration
- Burn-in/tuning length b and the self-tuning CosmoMC tuning endpoint (n = 2000)

## Outputs

- Array of epsilon values over the (L, n) grid
- Convergence plot (epsilon_convergence_curves.png) with a horizontal epsilon = 0.1 reference line
- Stopping decision: first sequential step count at which epsilon <= 0.1 for CosmoHammer

## How to apply

Retrieve the autocorrelation times tau from Table 2 of the CosmoHammer paper (arXiv:1212.1721) for the grid of walker counts L and sequential step counts n, fixing the burn-in length b and the self-tuning CosmoMC tuning endpoint at n = 2000 as specified in the benchmark. Estimate tau directly from chain data if unavailable, e.g. with the acor Python wrapper of Goodman's routine (acor.acor(x) returns tau, mean, sigma). Evaluate epsilon = |2*tau/((n-b)*L)|^(1/2) at every (L, n) grid point with numpy, then plot epsilon versus n for CosmoHammer and self-tuning CosmoMC on one axes with a horizontal reference line at epsilon = 0.1. The stopping decision is verified when the CosmoHammer curve drops to epsilon <= 0.1 at a sequential step count below the competitor's tuning endpoint (n = 2000); the finding is that even L = 10 walkers suffice.

## Related tools

- **Python** (Language in which the epsilon = |2*tau/((n-b)*L)|^(1/2) computation is evaluated with numpy and plotted with matplotlib)
- **acor** (Python wrapper estimating the autocorrelation time tau of MCMC time-series chains used in the epsilon formula) — https://github.com/dfm/acor
- **CosmoHammer** (Framework embedding the emcee affine-invariant ensemble sampler whose walker count L and step count n supply the (L, n) grid of the stopping analysis) — https://github.com/cosmo-ethz/CosmoHammer

## Examples

```
import acor; tau, mean, sigma = acor.acor(x); eps = (abs(2*tau/((n-b)*L)))**0.5
```

## Evaluation signals

- epsilon decreases monotonically in both n and L for fixed tau and b, as implied by the formula |2*tau/((n-b)*L)|^(1/2)
- At n = b the formula diverges; verify no grid point uses n <= b
- CosmoHammer's curve crosses epsilon = 0.1 at a sequential step count strictly below n = 2000
- Plot file epsilon_convergence_curves.png contains a horizontal reference line at epsilon = 0.1 and curves for both CosmoHammer and self-tuning CosmoMC

## Limitations

- epsilon scales with the inverse square root of (n-b)*L, so scaling saturates: with more processes than walkers the processes idle, limiting effective throughput gains
- The 10% threshold and the n = 2000 tuning endpoint are specific to the WMAP-likelihood benchmark in arXiv:1212.1721 and do not transfer to other likelihoods

## Evidence

- [other] Use Python (numpy) to evaluate epsilon = |2*tau/((n-b)*L)|^(1/2) at every (L, n) grid point.: [evidence span withheld — anchor status: not_found]
- [other] CosmoHammer reaches epsilon <= 0.1 before self-tuning CosmoMC finalises tuning: [evidence span withheld — anchor status: not_found]
- [readme] acor estimates the autocorrelation time of time series data: [evidence span withheld — anchor status: not_found]
- [readme] CosmoHammer embeds emcee: [evidence span withheld — anchor status: not_found]
