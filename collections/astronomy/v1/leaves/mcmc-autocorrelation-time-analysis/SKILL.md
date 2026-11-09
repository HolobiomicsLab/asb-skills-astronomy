---
name: mcmc-autocorrelation-time-analysis
description: Use when comparing sampler efficiency (e.g. emcee/CosmoHammer vs Metropolis-Hastings
  CosmoMC) by estimating autocorrelation times per parameter, or when determining
  the sampling effort needed to reach a target statistical error epsilon <= 0.1.
when_to_use_negative:
- Chains are not yet converged or burn-in b is unknown — epsilon computed from Eq.
  8 with a wrong b is meaningless
- The sampler is not a Goodman-Weare ensemble sampler and the Appendix C estimator
  assumptions (exponential autocorrelation) do not apply
- The goal is wall-time scaling analysis (e.g. 30 h on dual core to 16 min on 2048
  cores) rather than statistical error of the sample
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: acor
  role: Python port of Goodman's ACOR routine used to estimate the exponential autocorrelation
    time tau of each chain (returns tau, mean, sigma)
  repo: https://github.com/dfm/acor
- name: CosmoHammer
  role: Framework embedding emcee to generate the WMAP 7 MCMC chains whose autocorrelation
    times and epsilon are analysed
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: Python
  role: numpy evaluation of epsilon on the (L, n) grid and matplotlib plotting of
    convergence curves
- name: CosmoMC
  role: Metropolis-Hastings baseline sampler (fine-tuned and self-tuning configurations)
    whose tau values are compared against CosmoHammer's
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:c88d677d2d090b2911954fc4802478aa17d81e2e9f31a71451d49cc7210e708a
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
  - CAMB
  - CosmoHammer
  - CosmoMC
  - Python
  - acor
  - emcee
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mcmc-autocorrelation-time-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate per-parameter autocorrelation times (tau) of MCMC chains using the Goodman-Weare Appendix C estimator (acor package), and use them to quantify the statistical error epsilon = |2*tau/((n-b)*L)|^(1/2) as a function of walkers L, sequential steps n, and burn-in b.

## When to use

Use when comparing sampler efficiency (e.g. emcee/CosmoHammer vs Metropolis-Hastings CosmoMC) by estimating autocorrelation times per parameter, or when determining the sampling effort needed to reach a target statistical error epsilon <= 0.1.

## When NOT to use

- Chains are not yet converged or burn-in b is unknown — epsilon computed from Eq. 8 with a wrong b is meaningless
- The sampler is not a Goodman-Weare ensemble sampler and the Appendix C estimator assumptions (exponential autocorrelation) do not apply
- The goal is wall-time scaling analysis (e.g. 30 h on dual core to 16 min on 2048 cores) rather than statistical error of the sample

## Inputs

- MCMC chain time series per sampled cosmological parameter (e.g. H0, Omega_b, Omega_cdm, tau, n_s, logA, sigma8) from emcee/CosmoHammer and CosmoMC runs
- Grid of walker counts L and sequential step counts n, with burn-in length b and CosmoMC tuning endpoint n = 2000
- WMAP 7-year likelihood configuration (parameter start/min/max/width arrays, e.g. [[70, 40, 100, 3], ...])

## Outputs

- Table of per-parameter autocorrelation times tau for CosmoHammer, fine-tuned CosmoMC, and self-tuning CosmoMC (Table 2 equivalent, CSV)
- Plot epsilon versus n for CosmoHammer and self-tuning CosmoMC with epsilon = 0.1 reference line (epsilon_convergence_curves.png)

## How to apply

Run each sampler on the same parameter space — here WMAP 7-year likelihood with CAMB via CosmoHammer's emcee (Goodman-Weare affine-invariant) sampler with a walkers ratio of 50 (350 walkers for 7 parameters), plus fine-tuned and self-tuning CosmoMC Metropolis-Hastings baselines. For each chain, compute the exponential autocorrelation time per parameter using the acor package (github.com/dfm/acor), a Python port of Jonathan Goodman's ACOR routine, which returns tau, mean, and sigma from the time series. Then evaluate epsilon = |2*tau/((n-b)*L)|^(1/2) on a grid of walker counts L and sequential step counts n, using burn-in length b fixed by the paper's benchmark configuration (self-tuning CosmoMC tuning endpoint at n = 2000). Plot epsilon versus n for both samplers with a horizontal reference line at epsilon = 0.1 (matplotlib) and verify that CosmoHammer drops into the 10% error regime for as few as 10 walkers before n = 2000. Expected magnitudes: tau of roughly 43–48 for CosmoHammer versus 13–19 for CosmoMC, since the ensemble sampler incurs higher autocorrelation but far greater parallelism.

## Related tools

- **acor** (Python port of Goodman's ACOR routine used to estimate the exponential autocorrelation time tau of each chain (returns tau, mean, sigma)) — https://github.com/dfm/acor
- **CosmoHammer** (Framework embedding emcee to generate the WMAP 7 MCMC chains whose autocorrelation times and epsilon are analysed) — http://github.com/cosmo-ethz/CosmoHammer
- **Python** (numpy evaluation of epsilon on the (L, n) grid and matplotlib plotting of convergence curves)
- **CosmoMC** (Metropolis-Hastings baseline sampler (fine-tuned and self-tuning configurations) whose tau values are compared against CosmoHammer's)

## Examples

```
import acor; tau, mean, sigma = acor.acor(chain[:, param]); import numpy as np; eps = np.sqrt(2*tau/((n-b)*L))
```

## Evaluation signals

- Reproduced tau values fall in the Table 2 ranges: approximately 43.4–48.2 for CosmoHammer, 13.8–18.4 for fine-tuned CosmoMC, 13.3–19.1 for self-tuning CosmoMC
- CSV of tau values exists, parses, and contains CosmoHammer values around 44 and fine-tuned CosmoMC values around 15–18 per parameter
- CosmoHammer epsilon curve reaches epsilon <= 0.1 at a sequential step count below n = 2000, even for L = 10 walkers
- Figure epsilon_convergence_curves.png is produced with a horizontal reference line at epsilon = 0.1

## Limitations

- CosmoHammer's per-chain autocorrelation times (tau ~ 44) are about three times larger than Metropolis-Hastings CosmoMC (tau ~ 15–18), so error per walker converges more slowly per sequential step
- Scaling saturates once there are more processes than available walkers, causing processes and their spawned threads to idle (plateau observed at 64 nodes)
- acor requires only NumPy but needs python-dev and python-numpy-dev packages on some systems; WMAP data must be locally accessible and CFITSIO installed

## Evidence

- [discussion] Autocorrelation time estimation via acor on Goodman-Weare chains: [evidence span withheld — anchor status: not_found]
- [results] Statistical error formula and convergence benchmark: [evidence span withheld — anchor status: not_found]
- [readme] acor README usage: [evidence span withheld — anchor status: not_found]
