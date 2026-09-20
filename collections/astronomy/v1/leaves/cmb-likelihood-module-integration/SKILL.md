---
name: cmb-likelihood-module-integration
description: Reference for plugging cosmological core/likelihood modules (e.g. CambCoreModule,
  WmapLikelihoodModule) into CosmoHammer's chain when building MCMC parameter-estimation
  configurations over CMB data.
when_to_use_negative:
- When the task is sampler performance/scaling analysis (wall-time vs. cores) rather
  than wiring likelihood modules into a chain
- When the target is a non-CosmoHammer MCMC framework (e.g. raw emcee or CosmoMC)
  whose module-registration API differs
- When WMAP/Planck likelihood data are not accessible on the filesystem and CFITSIO
  cannot be installed, since the Fortran likelihood code cannot run
edam_operation: http://edamontology.org/operation_2426
tools:
- name: CosmoHammer
  role: Framework whose LikelihoodComputationChain stores and orders core/likelihood
    modules and enforces parameter bounds
  repo: https://github.com/cosmo-ethz/CosmoHammer
- name: CosmoHammerPlugins
  role: Collection of likelihood and core modules (PyCamb, WMAP, Planck) to install
    separately and register with the chain
  repo: http://github.com/cosmo-ethz/CosmoHammerPlugins
- name: CAMB
  role: Boltzmann integrator wrapped as CambCoreModule to write theory predictions
    into the chain context
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:5cac1439fa06917427958df1030123e7d753f43c7d34ca06f2cf128efeb42af9
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cmb-likelihood-module-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assemble self-contained core (CAMB theory prediction) and likelihood (WMAP) modules into CosmoHammer's LikelihoodComputationChain so they compose into different sampling configurations via ordered registration, shared chain context, and bounded parameter arrays.

## When to use

Reference for plugging cosmological core/likelihood modules (e.g. CambCoreModule, WmapLikelihoodModule) into CosmoHammer's chain when building MCMC parameter-estimation configurations over CMB data.

## When NOT to use

- When the task is sampler performance/scaling analysis (wall-time vs. cores) rather than wiring likelihood modules into a chain
- When the target is a non-CosmoHammer MCMC framework (e.g. raw emcee or CosmoMC) whose module-registration API differs
- When WMAP/Planck likelihood data are not accessible on the filesystem and CFITSIO cannot be installed, since the Fortran likelihood code cannot run

## Inputs

- numpy parameter array with columns [start centre, min, max, start width] (e.g. params = np.array([[70, 40, 100, 3], ...]))
- core module instance (e.g. CambCoreModule wrapping the CAMB Boltzmann integrator)
- likelihood module instance (e.g. WmapLikelihoodModule for WMAP 3/5/7/9-year likelihood data)
- WMAP likelihood data accessible on the local filesystem

## Outputs

- configured LikelihoodComputationChain with ordered core and likelihood modules
- numeric likelihood value returned by computeLikelihood(ctx) for a parameter point
- chain context (ctx) carrying theory predictions between core and likelihood modules

## How to apply

Define a numpy parameter array with columns [start centre, min, max, start width] (e.g. H0 = [70, 40, 100, 3]) and instantiate LikelihoodComputationChain with min=params[:,1], max=params[:,2] so sampled parameters are rejected outside physically motivated bounds. Register core modules with chain.addCoreModule(camb) and likelihood modules with chain.addLikelihoodModule(wmap.WmapLikelihoodModule()); the chain stores them and initialises them in the right order. Call chain.setup() so CAMB and WMAP modules are initialised before evaluation. During evaluation the core module writes its prediction into the shared chain context (ctx) and the likelihood module reads it via computeLikelihood(ctx). Modules for PyCamb, WMAP, Planck and more are installed separately from the cosmoHammerPlugins project.

## Related tools

- **CosmoHammer** (Framework whose LikelihoodComputationChain stores and orders core/likelihood modules and enforces parameter bounds) — https://github.com/cosmo-ethz/CosmoHammer
- **CosmoHammerPlugins** (Collection of likelihood and core modules (PyCamb, WMAP, Planck) to install separately and register with the chain) — http://github.com/cosmo-ethz/CosmoHammerPlugins
- **CAMB** (Boltzmann integrator wrapped as CambCoreModule to write theory predictions into the chain context)

## Examples

```
chain = LikelihoodComputationChain(min=params[:,1], max=params[:,2]); chain.addCoreModule(camb); chain.addLikelihoodModule(wmap.WmapLikelihoodModule()); chain.setup(); print(wmapLikelihood.computeLikelihood(ctx))
```

## Evaluation signals

- Script runs end-to-end with no parameter-bound violations against min=params[:,1] and max=params[:,2]
- chain.setup() successfully initialises both the CAMB and WMAP modules
- computeLikelihood(ctx) returns a numeric likelihood value (e.g. for WMAP 3/5/7/9-year data) rather than raising or returning None
- Core-module prediction is visible in the shared ctx object before the likelihood module reads it

## Limitations

- WMAP data must be accessible on the filesystem and CFITSIO must be installed; these external dependencies are not bundled
- Each cosmoHammerPlugins module must be installed separately
- The distribution was only tested with Intel's ifort compiler and mkl libraries; other Fortran compilers are untested

## Evidence

- [discussion] Define parameter start centre, min, max and start width arrays: "parameter start centre , min , max, start width params=np . array ([[70 , 40, 100, 3] ,"
- [results] Instantiate the chain with bounds and register core/likelihood modules: "chain.addCoreModule(camb)

chain.addLikelihoodModule(wmap9.WmapExtLikelihoodModule())

chain.setup()"
- [discussion] setup() initialises the registered modules: "By calling the setup() function the CAMB and WMAP modules are initialised."
- [readme] Pluggable likelihood modules are the design goal: "It gives the user the possibility to plug in modules for the computation of any desired likelihood."
- [intro] Extensibility by plugging in likelihood and theory modules: "it is straight forward to plug in the Python modules containing further likelihood functions or codes for"
