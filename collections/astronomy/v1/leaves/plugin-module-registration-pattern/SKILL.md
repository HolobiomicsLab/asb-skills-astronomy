---
name: plugin-module-registration-pattern
description: Use when building a cosmological (or any) MCMC likelihood pipeline on
  CosmoHammer and you need to plug in, order, and initialise custom core/likelihood
  modules with parameter-bound enforcement before sampling.
when_to_use_negative:
- When the likelihood/theory code cannot be wrapped as a Python module — CosmoHammer's
  plug-in architecture requires Python modules containing further likelihood functions
  or theory prediction codes
- When you need a monolithic sampler that bundles its own Boltzmann code and likelihood
  (e.g. CosmoMC) rather than composing standalone modules
- When sampling parameters should be allowed outside the physical min/max bounds supplied
  at chain construction
tools:
- name: CosmoHammer
  role: Framework providing LikelihoodComputationChain, addCoreModule()/addLikelihoodModule()
    registration, chain.setup(), the chain context, and the emcee-embedded sampler
  repo: https://github.com/cosmo-ethz/CosmoHammer
- name: CosmoHammerPlugins
  role: Collection of public core and likelihood modules (PyCamb, WMAP, Planck) that
    are installed separately and registered into the chain
  repo: http://github.com/cosmo-ethz/CosmoHammerPlugins
- name: CAMB
  role: Boltzmann integrator wrapped as CambCoreModule, the core module whose predictions
    populate the chain context
- name: emcee
  role: Affine invariant ensemble sampler embedded by CosmoHammer that consumes the
    chain's computed likelihood
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:4e450421518c5997f61a2f3e738a693c14369d9704a4fc3eb7b096a9aa386ba8
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# plugin-module-registration-pattern

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extends the CosmoHammer MCMC framework by registering self-contained core and likelihood modules (e.g. CambCoreModule and WmapLikelihoodModule) into a LikelihoodComputationChain, which stores them, initialises them in order via setup(), and shuttles data between them through a shared chain context (ctx).

## When to use

Use when building a cosmological (or any) MCMC likelihood pipeline on CosmoHammer and you need to plug in, order, and initialise custom core/likelihood modules with parameter-bound enforcement before sampling.

## When NOT to use

- When the likelihood/theory code cannot be wrapped as a Python module — CosmoHammer's plug-in architecture requires Python modules containing further likelihood functions or theory prediction codes
- When you need a monolithic sampler that bundles its own Boltzmann code and likelihood (e.g. CosmoMC) rather than composing standalone modules
- When sampling parameters should be allowed outside the physical min/max bounds supplied at chain construction

## Inputs

- numpy parameter array with columns [start centre, min, max, start width] (e.g. params=np.array([[70, 40, 100, 3], ...]))
- core module instance (e.g. CambCoreModule) wrapping a theory code such as CAMB
- likelihood module instance (e.g. wmap.WmapLikelihoodModule) with WMAP likelihood data accessible on the filesystem

## Outputs

- Initialised LikelihoodComputationChain with ordered core and likelihood modules
- Numeric likelihood value returned by computeLikelihood(ctx) per parameter point
- Context (ctx) object carrying theory predictions from core modules to likelihood modules

## How to apply

Define the sampling parameters as a numpy array with columns [start centre, min, max, start width] (e.g. H0 = [70, 40, 100, 3]), then instantiate LikelihoodComputationChain(min=params[:,1], max=params[:,2]) so out-of-bounds parameters are rejected during sampling. Register theory and likelihood components via chain.addCoreModule(camb) for the CAMB Boltzmann core module and chain.addLikelihoodModule(...) for WMAP likelihood modules. Call chain.setup() so the CAMB and WMAP modules are initialised before evaluation. During evaluation the core module writes its prediction into the shared chain context, and each likelihood module reads it via computeLikelihood(ctx), returning a numeric likelihood that the embedded emcee affine-invariant ensemble sampler consumes.

## Related tools

- **CosmoHammer** (Framework providing LikelihoodComputationChain, addCoreModule()/addLikelihoodModule() registration, chain.setup(), the chain context, and the emcee-embedded sampler) — https://github.com/cosmo-ethz/CosmoHammer
- **CosmoHammerPlugins** (Collection of public core and likelihood modules (PyCamb, WMAP, Planck) that are installed separately and registered into the chain) — http://github.com/cosmo-ethz/CosmoHammerPlugins
- **CAMB** (Boltzmann integrator wrapped as CambCoreModule, the core module whose predictions populate the chain context)
- **emcee** (Affine invariant ensemble sampler embedded by CosmoHammer that consumes the chain's computed likelihood)

## Examples

```
chain = LikelihoodComputationChain(min=params[:,1], max=params[:,2]); chain.addCoreModule(camb); chain.addLikelihoodModule(wmap.WmapLikelihoodModule()); chain.setup()
```

## Evaluation signals

- The script runs end-to-end after chain.setup() with both the CAMB and WMAP modules initialised
- computeLikelihood(ctx) returns a numeric likelihood value for a sampled parameter point
- No parameter-bound violations occur: points outside min=params[:,1] / max=params[:,2] are rejected by the chain
- Modules are registered in the correct order (core module added before likelihood modules) so the likelihood reads a populated ctx
- Additional modules (e.g. WMAP 3/5/7/9-year likelihoods) can be swapped in via addLikelihoodModule without changing the core module

## Limitations

- WMAP data must be accessible on the filesystem and CFITSIO has to be installed — this external dependency is not bundled
- Each CosmoHammerPlugins module must be installed separately (extract tarballs and run setup.py build/install per module)
- Likelihood modules are provided for specific WMAP data releases (3/5/7/9-year); extending to other likelihoods requires writing a new Python plug-in module

## Evidence
