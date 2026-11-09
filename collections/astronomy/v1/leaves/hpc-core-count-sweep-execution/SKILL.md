---
name: hpc-core-count-sweep-execution
description: Use when you need to measure or predict how the wall time of a parallelised
  sampling or simulation workflow scales with the number of cores/nodes, e.g. planning
  cloud (EC2) or grid budgets for MCMC runs.
when_to_use_negative:
- The target core count exceeds the walker budget (e.g. more processes than the 300
  walkers) — processes idle and the scaling measurement saturates, giving meaningless
  timings
- The goal is comparing sampler statistical quality (Goodman–Weare vs Metropolis–Hastings)
  rather than wall-time scalability — Metropolis–Hastings is dominated by overheads
  and this sweep only measures throughput scaling
- The application cannot be parallelised at the likelihood-evaluation level (the speedup
  relies on distributing independent chain likelihood evaluations across MPI ranks)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: CosmoHammer
  role: Framework whose CosmoHammerSampler (walkers ratio, burn-in, sample iterations,
    startSampling()) is run at each core count in the sweep
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Affine-invariant Goodman–Weare ensemble sampler embedded by CosmoHammer; its
    walker ensemble is the unit of parallel work distributed over cores
- name: mpi4py
  role: Python MPI wrapper used to distribute CosmoHammer across multiple nodes during
    the core-count sweep
- name: Python
  role: Implementation language of the framework and the least-squares log-log scaling
    fit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:7ff43bb999012ebd35eeb64d8474833e60f82937292a1f19f0a59a0726437bf4
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hpc-core-count-sweep-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for quantifying how MCMC wall time scales with core count: sweep over core counts (e.g. 1–2048 cores on EC2 or a cluster) running CosmoHammer/emcee likelihood sampling under MPI, record wall times, and fit log(T) vs log(N) to recover a power-law scaling exponent (alpha = 0.89 in the reference study).

## When to use

Use when you need to measure or predict how the wall time of a parallelised sampling or simulation workflow scales with the number of cores/nodes, e.g. planning cloud (EC2) or grid budgets for MCMC runs.

## When NOT to use

- The target core count exceeds the walker budget (e.g. more processes than the 300 walkers) — processes idle and the scaling measurement saturates, giving meaningless timings
- The goal is comparing sampler statistical quality (Goodman–Weare vs Metropolis–Hastings) rather than wall-time scalability — Metropolis–Hastings is dominated by overheads and this sweep only measures throughput scaling
- The application cannot be parallelised at the likelihood-evaluation level (the speedup relies on distributing independent chain likelihood evaluations across MPI ranks)

## Inputs

- Installed CosmoHammer framework with PyCamb and WMAP wrapper modules (tarballs built/installed via setup.py)
- mpi4py for multi-node MPI distribution
- WMAP 7 likelihood data accessible on the local filesystem with CFITSIO installed
- Parameter array [[start-centre, min, max, start-width], ...] for the 6 cosmological parameters (e.g. [[70, 40, 100, 3], ...])
- A benchmark sweep list of core counts (from dual-core up to 2048 cores)

## Outputs

- Results table of wall time vs core count for each benchmark run
- Fitted power-law scaling exponent alpha from log(T) vs log(N) least-squares fit (expected 0.89, i.e. 11% overhead per core doubling)
- Summary of achievable speedup, e.g. 30 h on dual-core notebook scaled to ~16 min on 2048 cores

## How to apply

Set up the full likelihood chain first (parameter arrays with start-centre/min/max/start-width, LikelihoodComputationChain with min=params[:,1] and max=params[:,2], CambCoreModule plus WMAP likelihood modules, then chain.setup()), and install mpi4py for multi-node MPI distribution. For each core count in the sweep, construct CosmoHammerSampler with a fixed walkers ratio of 50 (50 x 6 = 300 walkers for six cosmological parameters), fixed burn-in and sample iterations, and launch startSampling() under MPI so likelihood evaluations are distributed across ranks. Record the wall time and core count of each run into a results table. Fit log(T) vs log(N) by least squares to obtain the scaling exponent alpha; the reference study recovered alpha = 0.89, i.e. an 11% overhead per doubling of cores, scaling from 30 h on a dual-core notebook to 16 min on 2048 cores. Keep the core count within the walker budget (here ~300): once more processes than walkers exist, processes idle and scaling saturates.

## Related tools

- **CosmoHammer** (Framework whose CosmoHammerSampler (walkers ratio, burn-in, sample iterations, startSampling()) is run at each core count in the sweep) — http://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Affine-invariant Goodman–Weare ensemble sampler embedded by CosmoHammer; its walker ensemble is the unit of parallel work distributed over cores)
- **mpi4py** (Python MPI wrapper used to distribute CosmoHammer across multiple nodes during the core-count sweep)
- **Python** (Implementation language of the framework and the least-squares log-log scaling fit)

## Examples

```
sampler = CosmoHammerSampler(params=params, chain=chain, filePrefix='wmap7_', walkersRatio=50, burninIterations=100, sampleIterations=200); sampler.startSampling()  # run under MPI per core count
```

## Evaluation signals

- Fitted scaling exponent alpha agrees with 0.89 within reasonable tolerance (11% overhead per doubling of cores)
- Wall time at 2048 cores is approximately 16 min when scaled from the 30 h dual-core notebook baseline
- Wall-time trend is monotone decreasing over the achievable core range
- Core counts stay within the ~300-walker budget (walkers ratio 50 x 6 parameters); beyond this, idle-process saturation flattens the curve
- Each run uses an identical sampler configuration (walkers ratio, burn-in, sample iterations) so timings are comparable across core counts

## Limitations

- Scaling saturates once more processes are available than walkers; the study observed a plateau at 64 nodes as processes and their spawned threads idle
- Alpha = 0.89 implies non-ideal scaling — an 11% overhead per doubling of cores, so wall time does not halve on each doubling
- The software was only tested with Intel's ifort compiler and mkl libraries; other Fortran compilers are untested
-  mpi4py support is optional and only relevant when CosmoHammer is distributed on multiple nodes

## Evidence
