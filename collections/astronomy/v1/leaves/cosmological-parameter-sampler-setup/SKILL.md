---
name: cosmological-parameter-sampler-setup
description: Use when you need to sample a posterior over cosmological parameters
  (e.g. 6 WMAP-constrained CMB parameters) with emcee/CosmoHammer, requiring explicit
  parameter bounds, walker counts, burn-in/sample iterations, and chain module wiring.
when_to_use_negative:
- When the target is a Metropolis–Hastings sampler — the article reports Metropolis–Hastings
  is dominated by overheads in parallelised MCMC sampling, whereas the affine-invariant
  emcee ensemble is the efficient choice here
- When you need exact scalar likelihood values or posterior parameter tables as ground-truth
  outputs — the article's code snippets only print placeholder likelihood statements
  with no numeric results
- When you cannot provide WMAP data locally on the filesystem or install CFITSIO —
  the WMAP wrapper will fail since these external dependencies are not bundled
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3316
tools:
- name: CosmoHammer
  role: Python framework that embeds emcee and orchestrates the LikelihoodComputationChain,
    CosmoHammerSampler and MPI-distributed sampling setup
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Affine-invariant MCMC ensemble sampler by Goodman and Weare (2012) doing the
    actual posterior sampling
- name: CosmoHammerPlugins
  role: Collection of PyCamb, WMAP and Planck likelihood/core modules that must be
    installed separately and plugged into the chain
  repo: http://github.com/cosmo-ethz/CosmoHammerPlugins
- name: CAMB
  role: Boltzmann integrator whose CambCoreModule computes theory predictions inside
    the likelihood chain
- name: mpi4py
  role: Optional Python MPI wrapper used only to distribute the sampler across multiple
    nodes
- name: acor
  role: Port of Goodman's ACOR routine to estimate the autocorrelation time tau of
    the sampled time series; install via pip install acor or python setup.py install
  repo: https://github.com/dfm/acor
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:030ee785499f9936f5239fb26f7d050781256808b216087e58f9e21a50e0ba30
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
  - CosmoHammerPlugins
  - Python
  - acor
  - emcee
  - mpi4py
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cosmological-parameter-sampler-setup

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure and launch an emcee-based affine-invariant ensemble MCMC sampler for cosmological parameter estimation by assembling a LikelihoodComputationChain of CAMB core and WMAP likelihood modules in CosmoHammer, with a walkers-ratio of 50 (300 walkers for 6 parameters) and optional MPI distribution across cloud nodes.

## When to use

Use when you need to sample a posterior over cosmological parameters (e.g. 6 WMAP-constrained CMB parameters) with emcee/CosmoHammer, requiring explicit parameter bounds, walker counts, burn-in/sample iterations, and chain module wiring.

## When NOT to use

- When the target is a Metropolis–Hastings sampler — the article reports Metropolis–Hastings is dominated by overheads in parallelised MCMC sampling, whereas the affine-invariant emcee ensemble is the efficient choice here
- When you need exact scalar likelihood values or posterior parameter tables as ground-truth outputs — the article's code snippets only print placeholder likelihood statements with no numeric results
- When you cannot provide WMAP data locally on the filesystem or install CFITSIO — the WMAP wrapper will fail since these external dependencies are not bundled

## Inputs

- Parameter array of [start centre, min, max, start width] rows for the sampled cosmological parameters (e.g. [[70, 40, 100, 3], ...] for 6 parameters)
- PyCamb / WMAP wrapper modules (CosmoHammerPlugins) and locally available WMAP likelihood data with CFITSIO installed
- Chain configuration: walkersRatio (e.g. 50), burn-in iterations, sample iterations, file prefix
- Optional MPI environment (mpi4py) for multi-node distribution

## Outputs

- MCMC chain samples of the posterior over the cosmological parameters, persisted under the chosen file prefix
- Log-likelihood trajectories per WMAP likelihood module evaluated at each step
- Autocorrelation-time estimates for the sampled chains via the acor module (tau, mean, sigma)

## How to apply

Define a parameter array of [start centre, min, max, start width] rows (e.g. [[70, 40, 100, 3], ...] for the 6 cosmological parameters) and instantiate LikelihoodComputationChain(min=params[:,1], max=params[:,2]) so the bounds reject proposals outside the physical range. Add the CambCoreModule as core module plus the WMAP likelihood modules (e.g. wmap9.WmapLikelihoodModule / WmapExtLikelihoodModule), then call chain.setup() to initialise CAMB and WMAP. Create CosmoHammerSampler with a walkersRatio of 50 (50 x 6 = 300 walkers for 6 parameters), chosen burn-in and sample iterations, and a file prefix for chain output, then call startSampling(); under MPI the chain's likelihood evaluations are distributed across ranks. Optionally first locate the best-fit point with ParticleSwarmOptimizer before MCMC sampling to start near high likelihood. Keep core counts within the ~300-walker budget, because once more processes are available than walkers, idle processes cause a scaling plateau (e.g. at 64 nodes).

## Related tools

- **CosmoHammer** (Python framework that embeds emcee and orchestrates the LikelihoodComputationChain, CosmoHammerSampler and MPI-distributed sampling setup) — http://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Affine-invariant MCMC ensemble sampler by Goodman and Weare (2012) doing the actual posterior sampling)
- **CosmoHammerPlugins** (Collection of PyCamb, WMAP and Planck likelihood/core modules that must be installed separately and plugged into the chain) — http://github.com/cosmo-ethz/CosmoHammerPlugins
- **CAMB** (Boltzmann integrator whose CambCoreModule computes theory predictions inside the likelihood chain)
- **mpi4py** (Optional Python MPI wrapper used only to distribute the sampler across multiple nodes)
- **acor** (Port of Goodman's ACOR routine to estimate the autocorrelation time tau of the sampled time series; install via pip install acor or python setup.py install) — https://github.com/dfm/acor

## Examples

```
chain = LikelihoodComputationChain(min=params[:,1], max=params[:,2]); chain.addCoreModule(camb); chain.addLikelihoodModule(wmap9.WmapExtLikelihoodModule()); chain.setup(); sampler.startSampling()
```

## Evaluation signals

- Chain.setup() must initialise CAMB and the WMAP likelihood modules without errors, confirming WMAP data and CFITSIO are accessible on the filesystem
- Walker count must equal walkersRatio x number of parameters (e.g. 50 x 6 = 300 walkers) — verify sampler logs report this ensemble size
- Core count must stay within the ~300-walker budget; a plateau in wall-time scaling (e.g. at 64 nodes) indicates idle processes from oversubscription
- Fitted log(T) vs log(N) scaling exponent should reproduce alpha = 0.89 (11% overhead per core doubling) when benchmarking scaled-out runs
- Sampling should terminate with the expected 'done!' state and persisted chain files under the given file prefix after the configured burn-in and sample iterations

## Limitations

- Only tested with Intel's ifort compiler and mkl libraries; other Fortran compilers are untested
- WMAP data must be locally accessible on the filesystem and CFITSIO must be installed — not bundled with the distribution
- mpi4py support is optional and only used when distributing across multiple nodes; single-node runs do not need it
- Scaling saturates: once there are more processes than walkers, processes and their spawned threads idle (plateau observed at 64 nodes)
- Distribution tested only with the Intel ifort/mkl toolchain and tarballs are hosted on an ETH Zurich personal page rather than a versioned archive

## Evidence

- [readme] CosmoHammer embeds emcee with affine-invariant ensemble sampling and pluggable likelihood modules for chain setup: "CosmoHammer is a framework which embeds `emcee <http://arxiv.org/abs/1202.3665>`_ , an implementation by Foreman-Mackey et al. (2012) of the `Affine Invariant Markov chain Monte Carlo (MCMC) Ensemble"
- [discussion] LikelihoodComputationChain is instantiated with min/max bounds from the parameter array: [evidence span withheld — anchor status: not_found]
- [discussion] chain.setup() initialises the CAMB and WMAP modules: "By calling the setup() function the CAMB and WMAP modules are initialised. chain . setup ()"
- [discussion] WMAP data must be locally accessible and CFITSIO installed: [evidence span withheld — anchor status: not_found]
