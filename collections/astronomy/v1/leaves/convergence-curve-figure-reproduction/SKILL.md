---
name: convergence-curve-figure-reproduction
description: Use when reproducing or benchmarking MCMC statistical error epsilon as
  a function of walker count L, sequential steps n, burn-in b, and autocorrelation
  time tau from Table 2 of a sampling study.
when_to_use_negative:
- The goal is estimating tau itself from raw chain time series rather than using published
  Table 2 values (that is the acor estimator's job, not this reproduction)
- The task is wall-clock scalability/performance benchmarking (e.g. 2048-core scaling
  or MPI overhead analysis) rather than statistical-error-vs-sampling-effort convergence
- Comparing a sampler other than CosmoHammer/emcee and Metropolis-Hastings CosmoMC
  without analogous tau and tuning-endpoint measurements
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_2258
tools:
- name: Python
  role: numpy/matplotlib evaluation of Eq. 8 and plotting of the epsilon-versus-n
    curves
- name: CosmoHammer
  role: the emcee-based ensemble MCMC framework whose convergence curve ( walkers,
    burn-in, sequential steps) is being evaluated against CosmoMC
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: acor
  role: Python autocorrelation-time estimator used to obtain tau values from chain
    time series if Table 2 values are not supplied directly
  repo: https://github.com/dfm/acor
- name: CosmoMC
  role: reference self-tuning Metropolis-Hastings sampler whose tuning endpoint at
    n = 2000 defines the comparison line
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:45559560ab71fa147a6cbb40c0ec343abeab1f8940b3b6392e5a943c76ad7ae8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cosmo-ethz/CosmoHammer
  tools:
  - CosmoHammer
  - CosmoMC
  - Python
  - acor
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# convergence-curve-figure-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the CosmoHammer paper's statistical-error convergence analysis (Eq. 8, Fig. 3 of arXiv:1212.1721) by evaluating epsilon = |2*tau/((n-b)*L)|^(1/2) over grids of walkers, sequential steps, and burn-in, and plotting when CosmoHammer reaches the 10% error regime versus self-tuning CosmoMC.

## When to use

Use when reproducing or benchmarking MCMC statistical error epsilon as a function of walker count L, sequential steps n, burn-in b, and autocorrelation time tau from Table 2 of a sampling study.

## When NOT to use

- The goal is estimating tau itself from raw chain time series rather than using published Table 2 values (that is the acor estimator's job, not this reproduction)
- The task is wall-clock scalability/performance benchmarking (e.g. 2048-core scaling or MPI overhead analysis) rather than statistical-error-vs-sampling-effort convergence
- Comparing a sampler other than CosmoHammer/emcee and Metropolis-Hastings CosmoMC without analogous tau and tuning-endpoint measurements

## Inputs

- Autocorrelation times tau from Table 2 of arXiv:1212.1721 for the grid of walker counts L and sequential steps n
- Benchmark burn-in/tuning length b and CosmoMC tuning endpoint n = 2000 from the paper's configuration
- Grid of walker counts L and sequential step counts n (e.g. L = 10 walkers)

## Outputs

- epsilon_convergence_curves.png: matplotlib figure of epsilon versus n for CosmoHammer and self-tuning CosmoMC with a reference line at epsilon = 0.1
- Per-(L, n) epsilon values from Eq. 8 (numpy array/table)

## How to apply

Retrieve the autocorrelation times tau from Table 2 of arXiv:1212.1721 for each (L, n) grid point, using the acor module (a port of Goodman's ACOR estimator) if recomputing tau from chains. Fix the burn-in/tuning length b and set the self-tuning CosmoMC tuning endpoint at n = 2000 as in the paper's benchmark. With numpy, evaluate epsilon = |2*tau/((n-b)*L)|^(1/2) (Eq. 8) at every grid point, then plot epsilon versus n for CosmoHammer and self-tuning CosmoMC on one matplotlib axes with a horizontal reference line at epsilon = 0.1. The target result is that even with only 10 walkers, CosmoHammer's curve crosses epsilon <= 0.1 before n = 2000, i.e. before CosmoMC finalises its proposal tuning; save the plot as epsilon_convergence_curves.png.

## Related tools

- **Python** (numpy/matplotlib evaluation of Eq. 8 and plotting of the epsilon-versus-n curves)
- **CosmoHammer** (the emcee-based ensemble MCMC framework whose convergence curve ( walkers, burn-in, sequential steps) is being evaluated against CosmoMC) — http://github.com/cosmo-ethz/CosmoHammer
- **acor** (Python autocorrelation-time estimator used to obtain tau values from chain time series if Table 2 values are not supplied directly) — https://github.com/dfm/acor
- **CosmoMC** (reference self-tuning Metropolis-Hastings sampler whose tuning endpoint at n = 2000 defines the comparison line)

## Examples

```
import numpy as np; import matplotlib.pyplot as plt; eps = np.sqrt(np.abs(2*tau/((n-b)*L)))  # Eq. 8, arXiv:1212.1721
```

## Evaluation signals

- CosmoHammer curve reaches epsilon <= 0.1 at n < 2000, before the self-tuning CosmoMC run finalises tuning at n = 2000, even for L = 10 walkers
- epsilon values satisfy epsilon^2 = 2*tau/((n-b)*L) exactly when recomputed from Table 2 tau values
- Figure contains a horizontal reference line at epsilon = 0.1 separating the 10% error regime
- Output file epsilon_convergence_curves.png is produced and shows both CosmoHammer and CosmoMC curves on a single axes
- Burn-in b is subtracted from n in the denominator as in Eq. 8, not omitted

## Limitations

- The analysis inherits the tau values of the paper's benchmark; different likelihoods (e.g. Planck instead of WMAP 7) will shift the convergence crossing points
- Scaling saturates once more processes exist than walkers, which is unrelated to epsilon but constrains how far L can be pushed in wall-clock terms
- This is an analytic convergence model (Eq. 8); it does not capture non-Gaussian chain behaviour beyond the autocorrelation-time assumption

## Evidence

- [discussion] epsilon formula at every grid point: "more processes are available than walkers to process, which in turn causes the processes and their spawned threads to idle"
- [readme] autocorrelation time estimation tooling: [evidence span withheld — anchor status: not_found]
- [abstract] emcee sampler identity: "we developed a Python framework called CosmoHammer which embeds emcee"
- [discussion] benchmark likelihood context: [evidence span withheld — anchor status: not_found]
