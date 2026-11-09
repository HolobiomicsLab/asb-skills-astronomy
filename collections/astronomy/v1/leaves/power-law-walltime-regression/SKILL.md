---
name: power-law-walltime-regression
description: Use when benchmarking how wall time of a parallelised sampler (e.g. CosmoHammer
  with emcee on EC2 or a grid) scales with core count, and you need a fitted power-law
  exponent alpha to quantify scaling overhead.
when_to_use_negative:
- 'The core count exceeds the number of walkers (here >300 for a 6-parameter run with
  walkers ratio 50): extra processes idle and wall-time scaling saturates, so a log-log
  fit over the saturated region would misrepresent the exponent.'
- The sampling method is Metropolis–Hastings, which the paper reports is dominated
  by overheads in parallelised MCMC sampling rather than benefiting from ensemble
  parallelism.
- The goal is sampling accuracy or posterior quality rather than computational scaling;
  this skill only characterises wall-time behaviour, and no numeric likelihood or
  posterior results are reported in the article.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CosmoHammer
  role: Framework whose startSampling() runs are benchmarked across core counts to
    measure wall-time scaling
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Affine invariant ensemble sampler embedded by CosmoHammer whose parallelisability
    is being characterised
- name: mpi4py
  role: Python MPI wrapper used to distribute CosmoHammer sampling across multiple
    nodes during the core-count sweep
- name: CAMB
  role: Boltzmann integrator supplying likelihood evaluations via CambCoreModule within
    the benchmarked chain
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:081b9cc86e2246bf07b6de0641ddfdd01a3d060a4a9d75e2b1bdbb339805c809
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cosmo-ethz/CosmoHammer
  tools:
  - CAMB
  - CosmoHammer
  - Python
  - emcee
  - mpi4py
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# power-law-walltime-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark wall time of a parallelised MCMC sampling run across a sweep of core counts and fit log(T) vs log(N) by least squares to recover the power-law scaling exponent alpha, following the CosmoHammer EC2 analysis that yielded alpha = 0.89 (11% overhead per doubling of cores).

## When to use

Use when benchmarking how wall time of a parallelised sampler (e.g. CosmoHammer with emcee on EC2 or a grid) scales with core count, and you need a fitted power-law exponent alpha to quantify scaling overhead.

## When NOT to use

- The core count exceeds the number of walkers (here >300 for a 6-parameter run with walkers ratio 50): extra processes idle and wall-time scaling saturates, so a log-log fit over the saturated region would misrepresent the exponent.
- The sampling method is Metropolis–Hastings, which the paper reports is dominated by overheads in parallelised MCMC sampling rather than benefiting from ensemble parallelism.
- The goal is sampling accuracy or posterior quality rather than computational scaling; this skill only characterises wall-time behaviour, and no numeric likelihood or posterior results are reported in the article.

## Inputs

- Parameter definition array (start centre, min, max, start width) for the 6 sampled cosmological parameters
- LikelihoodComputationChain with CambCoreModule and WMAP likelihood modules (chain.setup() initialised)
- CosmoHammerSampler configuration: walkers ratio 50 (300 walkers), burn-in and sample iterations
- Benchmark sweep of core counts (e.g. dual core notebook up to 2048 EC2 cores) with recorded wall times

## Outputs

- Results table of wall time vs core count for each benchmark run
- Least-squares fitted power-law exponent alpha from log(T) vs log(N) (reference: alpha = 0.89)
- Scaling-overhead percentage per doubling of cores (reference: 11%)

## How to apply

Set up the likelihood chain (parameter arrays such as [[70, 40, 100, 3], ...] with start-centre/min/max/start-width, LikelihoodComputationChain with CambCoreModule and WMAP likelihood modules, chain.setup()) and construct CosmoHammerSampler with a walkers ratio of 50 (50 x 6 = 300 walkers for 6 cosmological parameters). For each core count in the benchmark sweep, launch startSampling() under mpi4py MPI and record wall time together with core count into a results table. Fit log(T) vs log(N) by least squares to obtain the exponent alpha; the CosmoHammer reference value is alpha = 0.89, corresponding to an 11% scaling overhead when the number of cores is doubled. Keep the core count within the ~300-walker budget, because once more processes are available than walkers, processes and their spawned threads idle and the curve saturates (plateau observed at 64 nodes). Validate by checking the fitted alpha agrees with 0.89, that the wall time at 2048 cores is approximately 16 min (scaled from 30 h on a dual core notebook), and that the trend is monotone over the achievable core range.

## Related tools

- **CosmoHammer** (Framework whose startSampling() runs are benchmarked across core counts to measure wall-time scaling) — http://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Affine invariant ensemble sampler embedded by CosmoHammer whose parallelisability is being characterised)
- **mpi4py** (Python MPI wrapper used to distribute CosmoHammer sampling across multiple nodes during the core-count sweep)
- **CAMB** (Boltzmann integrator supplying likelihood evaluations via CambCoreModule within the benchmarked chain)

## Examples

```
sampler = CosmoHammerSampler(params=params, chain=chain, walkersRatio=50, burninIterations=..., sampleIterations=...); sampler.startSampling()
```

## Evaluation signals

- Fitted log-log slope alpha agrees with the published value 0.89 within reasonable tolerance
- Wall time at 2048 cores is approximately 16 min, consistent with scaling from 30 h on a dual core notebook
- Wall-time vs core-count trend is monotone over the achievable core range
- Core counts stay within the ~300-walker budget so idle-process saturation is avoided
- Each recorded (core count, wall time) point comes from a run launched with startSampling() under MPI with identical sampler settings (walkers ratio 50, same burn-in and sample iterations)

## Limitations

- Scaling saturates once more processes are available than walkers, causing processes and their spawned threads to idle (plateau observed at 64 nodes)
- The 11% overhead per core doubling (alpha = 0.89) is not perfect linear scaling in wall time
- Only tested with Intel's ifort compiler and mkl libraries; other Fortran compilers untested
- WMAP data must be accessible on the filesystem and CFITSIO installed; mpi4py is optional and only used for multi-node distribution
- No numeric likelihood values or posterior results are reported; the article characterises performance, not inference output

## Evidence

- [results] print("start sampling")
sampler.startSampling()
print("done!"): "print("start sampling")
sampler.startSampling()
print("done!")"
