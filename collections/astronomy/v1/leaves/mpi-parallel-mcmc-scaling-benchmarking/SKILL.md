---
name: mpi-parallel-mcmc-scaling-benchmarking
description: Use when you need to quantify how wall time of an MCMC likelihood chain
  (e.g. CAMB + WMAP likelihood) scales with core count on a cluster, cloud (EC2),
  or grid, and to measure parallelisation overhead.
when_to_use_negative:
- The number of MPI processes exceeds the number of ensemble walkers (e.g. >300 walkers
  for a ratio of 50 with 6 parameters) — extra processes idle and the scaling curve
  plateaus rather than informs
- The goal is posterior inference or likelihood values themselves — this skill benchmarks
  wall-time scaling and reports no numeric likelihoods or posterior results
- Serial or embarrassingly parallel workloads without an ensemble MCMC structure —
  Metropolis–Hastings is dominated by overheads and will not exhibit the same scalability
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3360
tools:
- name: CosmoHammer
  role: Framework that embeds emcee and distributes the likelihood chain (CambCoreModule
    + WMAP likelihood modules) across MPI ranks for the scaling benchmark
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Affine invariant ensemble sampler (Goodman & Weare) whose walker ensemble
    is what gets distributed across cores
- name: mpi4py
  role: Python MPI wrapper, optional, used only when Cosmo Hammer is distributed on
    multiple nodes
- name: CAMB
  role: Boltzmann integrator wrapped as CambCoreModule providing the likelihood evaluations
    being benchmarked
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:225858efcf65aa590f2dd0e289fa4047c98f792a2ad39d94c45eda841764aac5
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
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# MPI-parallel MCMC scaling benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the wall-time scaling of MPI-parallelised MCMC ensemble sampling (CosmoHammer/embedding emcee) across increasing core counts and fit a log-log power law to obtain the scaling exponent alpha (e.g. alpha = 0.89, an 11% overhead per doubling of cores).

## When to use

Use when you need to quantify how wall time of an MCMC likelihood chain (e.g. CAMB + WMAP likelihood) scales with core count on a cluster, cloud (EC2), or grid, and to measure parallelisation overhead.

## When NOT to use

- The number of MPI processes exceeds the number of ensemble walkers (e.g. >300 walkers for a ratio of 50 with 6 parameters) — extra processes idle and the scaling curve plateaus rather than informs
- The goal is posterior inference or likelihood values themselves — this skill benchmarks wall-time scaling and reports no numeric likelihoods or posterior results
- Serial or embarrassingly parallel workloads without an ensemble MCMC structure — Metropolis–Hastings is dominated by overheads and will not exhibit the same scalability

## Inputs

- CosmoHammer, PyCamb and WMAP wrapper tarballs (built via setup.py)
- WMAP likelihood data (e.g. WMAP 7-year) accessible on the local filesystem, with CFITSIO installed
- Parameter array of start centre, min, max, start width for 6 cosmological parameters (e.g. params = np.array([[70, 40, 100, 3], ...]))
- LikelihoodComputationChain configured with CambCoreModule and WMAP likelihood modules
- Sweep of MPI core counts (up to 2048 cores), bounded by the walker budget (walkers ratio 50 x 6 = 300 walkers)
- mpi4py for multi-node MPI distribution

## Outputs

- Results table of wall time vs core count for each benchmark run
- Least-squares fitted power-law exponent alpha from log(T) vs log(N) (expected alpha = 0.89)
- Scaling overhead per doubling of cores (11%) and endpoint wall times (30 h dual-core notebook vs 16 min on 2048 cores)

## How to apply

Install CosmoHammer, PyCamb and WMAP wrappers via setup.py build/install, with mpi4py for multi-node MPI distribution, WMAP data on the filesystem and CFITSIO installed. Define the parameter arrays (start centre, min, max, start width, e.g. [[70, 40, 100, 3], ...] for 6 cosmological parameters), build a LikelihoodComputationChain with min=params[:,1], max=params[:,2], add the CambCoreModule and WMAP likelihood modules, and call chain.setup(). For each core count in the sweep, instantiate CosmoHammerSampler with a walkers ratio of 50 (50 x 6 = 300 walkers) plus burn-in and sample iterations, run startSampling() under MPI, and record wall time and core count. Keep core counts within the ~300-walker budget: once more processes than walkers exist, processes idle and scaling saturates (plateau observed at 64 nodes). Finally, fit log(T) vs log(N) by least squares; the paper expects alpha = 0.89, i.e. an 11% scaling overhead per doubling of cores, and 16 min on 2048 cores versus 30 h on a dual core notebook.

## Related tools

- **CosmoHammer** (Framework that embeds emcee and distributes the likelihood chain (CambCoreModule + WMAP likelihood modules) across MPI ranks for the scaling benchmark) — http://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Affine invariant ensemble sampler (Goodman & Weare) whose walker ensemble is what gets distributed across cores)
- **mpi4py** (Python MPI wrapper, optional, used only when Cosmo Hammer is distributed on multiple nodes)
- **CAMB** (Boltzmann integrator wrapped as CambCoreModule providing the likelihood evaluations being benchmarked)

## Examples

```
chain = LikelihoodComputationChain(min=params[:,1], max=params[:,2]); chain.addCoreModule(camb); chain.addLikelihoodModule(wmap7.WmapLikelihoodModule()); chain.setup(); sampler = CosmoHammerSampler(params, chain, filePrefix, walkersRatio=50, burninIterations=..., sampleIterations=...); sampler.startSampling()
```

## Evaluation signals

- The least-squares fit of log(T) vs log(N) yields alpha ≈ 0.89 (11% scaling overhead per doubling of cores) within reasonable tolerance
- Wall time at 2048 cores is approximately 16 min, consistent with scaling from 30 h on a dual core notebook
- The wall-time trend is monotone decreasing over the achievable core range
- Scaling remains meaningful only while core count stays within the ~300-walker budget (walkers ratio 50 x 6); beyond it, processes idle and the curve plateaus (as seen at 64 nodes)

## Limitations

- Scaling saturates: once more processes are available than walkers, processes and their spawned threads idle, producing a plateau (observed at 64 nodes)
- mpi4py support is optional and only used for multi-node distribution; without it the benchmark is limited to a single node
- The current distribution has only been tested with Intel's ifort compiler and mkl libraries; other Fortran compilers are untested
- WMAP data must be locally accessible on the filesystem and CFITSIO must be installed (external, unbundled dependency)

## Evidence
