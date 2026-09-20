---
name: likelihood-chain-architecture-implementation
description: Use when assembling self-contained core modules (e.g., CAMB) and likelihood
  modules (e.g., WMAP) into a chain that shares a chain context (ctx) and enforces
  min/max parameter bounds for MCMC or PSO sampling.
when_to_use_negative:
- When the sampling infrastructure already exists (e.g., CosmoMC or another MCMC driver
  that directly combines CAMB with the likelihood) — this skill is for building/extending
  the CosmoHammer chain architecture
- When the WMAP data is not locally accessible on the filesystem and CFITSIO is not
  installed, since the WMAP likelihood modules cannot be initialised
- When you only need the emcee sampler itself without pluggable likelihood/core modules
  — plain emcee suffices
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CosmoHammer
  role: Provides the LikelihoodComputationChain class with addCoreModule(), addLikelihoodModule(),
    setup(), and the ctx object used to assemble core and likelihood modules
  repo: https://github.com/cosmo-ethz/CosmoHammer
- name: CAMB
  role: Boltzmann integrator wrapped by CambCoreModule; its predictions are written
    into the shared chain context
- name: emcee
  role: Affine invariant ensemble sampler embedded by CosmoHammer; consumes the assembled
    chain during startSampling()
- name: CosmoHammerPlugins
  role: Collection of public core and likelihood modules (PyCamb, WMAP, Planck) to
    register with the chain; each module installed separately
  repo: http://github.com/cosmo-ethz/CosmoHammerPlugins
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:330c87469ea83dcadc4fd1c4534d78436f7185aee0ff927535f58936b4743303
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cosmo-ethz/CosmoHammer
  repo_url: https://github.com/cosmo-ethz/CosmoHammer
  tools:
  - CAMB
  - CosmoHammer
  - CosmoHammerPlugins
  - Python
  - emcee
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# likelihood-chain-architecture-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the LikelihoodComputationChain control structure of CosmoHammer, which stores ordered core and likelihood modules (e.g., CambCoreModule and WMAP likelihood modules), initialises them via setup(), and enforces physically motivated parameter bounds during sampling.

## When to use

Use when assembling self-contained core modules (e.g., CAMB) and likelihood modules (e.g., WMAP) into a chain that shares a chain context (ctx) and enforces min/max parameter bounds for MCMC or PSO sampling.

## When NOT to use

- When the sampling infrastructure already exists (e.g., CosmoMC or another MCMC driver that directly combines CAMB with the likelihood) — this skill is for building/extending the CosmoHammer chain architecture
- When the WMAP data is not locally accessible on the filesystem and CFITSIO is not installed, since the WMAP likelihood modules cannot be initialised
- When you only need the emcee sampler itself without pluggable likelihood/core modules — plain emcee suffices

## Inputs

- numpy parameter array with columns [start centre, min, max, start width] (e.g., H0 = [70, 40, 100, 3])
- core module instance (e.g., CambCoreModule) computing theory predictions
- likelihood module instance (e.g., WmapLikelihoodModule, WmapExtLikelihoodModule) with computeLikelihood(ctx)
- WMAP likelihood data accessible on the filesystem (with CFITSIO installed)

## Outputs

- Initialised LikelihoodComputationChain with ordered core and likelihood modules and parameter bounds
- Numeric likelihood value returned by computeLikelihood(ctx) for a given parameter point
- Chain context (ctx) carrying core-module predictions to likelihood modules

## How to apply

Define the sampling parameter array in numpy with columns [start centre, min, max, start width] (e.g., H0 = [70, 40, 100, 3]). Instantiate LikelihoodComputationChain with min=params[:,1] and max=params[:,2] so out-of-bound parameters are rejected during sampling. Implement ordered storage for core and likelihood modules exposed via addCoreModule() and addLikelihoodModule(), plus a chain context (ctx) object that carries data between modules — the core module writes its prediction into ctx and the likelihood module reads it via computeLikelihood(ctx). Register CambCoreModule as a core module and the WMAP likelihood module (e.g., wmap.WmapLikelihoodModule() or wmap9.WmapExtLikelihoodModule()) as likelihood modules, then call chain.setup() so the CAMB and WMAP modules are initialised before evaluation. The same chain can serve both MCMC sampling (via CosmoHammerSampler) and particle swarm optimisation (ParticleSwarmOptimizer) for finding a best-fit point before sampling.

## Related tools

- **CosmoHammer** (Provides the LikelihoodComputationChain class with addCoreModule(), addLikelihoodModule(), setup(), and the ctx object used to assemble core and likelihood modules) — https://github.com/cosmo-ethz/CosmoHammer
- **CAMB** (Boltzmann integrator wrapped by CambCoreModule; its predictions are written into the shared chain context)
- **emcee** (Affine invariant ensemble sampler embedded by CosmoHammer; consumes the assembled chain during startSampling())
- **CosmoHammerPlugins** (Collection of public core and likelihood modules (PyCamb, WMAP, Planck) to register with the chain; each module installed separately) — http://github.com/cosmo-ethz/CosmoHammerPlugins

## Examples

```
chain = LikelihoodComputationChain(min=params[:,1], max=params[:,2]); chain.addCoreModule(CambCoreModule()); chain.addLikelihoodModule(wmap.WmapLikelihoodModule()); chain.setup(); wmapLikelihood.computeLikelihood(ctx)
```

## Evaluation signals

- chain.setup() initialises both the CAMB and WMAP modules without errors
- computeLikelihood(ctx) returns a numeric likelihood value with no parameter-bound violations
- Script runs end-to-end from parameter array definition through module registration to chain setup
- Out-of-bound parameters (outside params[:,1]..params[:,2]) are rejected by the chain's min/max boundaries
- The same chain instance can drive both CosmoHammerSampler.startSampling() and ParticleSwarmOptimizer best-fit search

## Limitations

- WMAP data must be locally accessible on the filesystem and CFITSIO has to be installed — an external dependency not bundled
- Public modules such as PyCamb, WMAP and Planck live in the separate cosmoHammerPlugins project and must be installed individually
- The current distribution has only been tested with Intel's ifort compiler and mkl libraries; other Fortran compilers are untested

## Evidence

- [results] chain.addLikelihoodModule(wmap9.WmapExtLikelihoodModule())

chain.setup(): "chain.addLikelihoodModule(wmap9.WmapExtLikelihoodModule())"
- [intro] The architecture of our code makes Cosmo Hammer easily extendable to other applications, as it is straight forward to plug in the Python modules containing further likelihood functions or codes for: [evidence span withheld — anchor status: not_found]
