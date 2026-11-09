---
name: mcmc-context-data-exchange-design
description: Use when building an extensible MCMC framework that must combine independent
  theory-prediction (core) and likelihood modules via a shared context, or when refactoring
  a monolithic likelihood computation into plug-in modules.
when_to_use_negative:
- The likelihood computation is a single self-contained function with no theory/likelihood
  split — a plain callable passed to the sampler is simpler than a chain of modules
- The sampler used is not emcee-based and already provides its own module/plugin interface
  (e.g. CosmoMC's own module system)
- You only need to run an existing CosmoHammer plugin unchanged — use the cosmoHammerPlugins
  modules directly rather than redesigning the chain
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3452
tools:
- name: CosmoHammer
  role: Framework providing LikelihoodComputationChain, addCoreModule()/addLikelihoodModule()
    registries, chain.setup(), and the shared ctx for module data exchange
  repo: https://github.com/cosmo-ethz/CosmoHammer
- name: CAMB
  role: Boltzmann integrator wrapped as CambCoreModule; writes theory predictions
    into the chain context for downstream likelihood modules
- name: cosmoHammerPlugins
  role: Collection of public core and likelihood modules (PyCamb, WMAP, Planck) that
    plug into the chain
  repo: http://github.com/cosmo-ethz/CosmoHammerPlugins
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:35007dfdd61c1339d9433c7d1a31196fc098999edee088f325c19d57aca15a7a
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
  - Python
  - cosmoHammerPlugins
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mcmc-context-data-exchange-design

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Designs a modular MCMC likelihood pipeline (as in CosmoHammer's LikelihoodComputationChain) where self-contained core modules (e.g. CAMB theory predictions) and likelihood modules (e.g. WMAP) exchange data via a shared chain context (ctx) object, assembled in a defined order with parameter bounds enforced.

## When to use

Use when building an extensible MCMC framework that must combine independent theory-prediction (core) and likelihood modules via a shared context, or when refactoring a monolithic likelihood computation into plug-in modules.

## When NOT to use

- The likelihood computation is a single self-contained function with no theory/likelihood split — a plain callable passed to the sampler is simpler than a chain of modules
- The sampler used is not emcee-based and already provides its own module/plugin interface (e.g. CosmoMC's own module system)
- You only need to run an existing CosmoHammer plugin unchanged — use the cosmoHammerPlugins modules directly rather than redesigning the chain

## Inputs

- numpy parameter array with columns [start centre, min, max, start width]
- core module implementing theory prediction (e.g. CambCoreModule wrapping CAMB)
- likelihood module implementing computeLikelihood(ctx) (e.g. WmapLikelihoodModule / WMAP 7-year likelihood code)
- parameter min/max boundary vectors for the chain

## Outputs

- configured LikelihoodComputationChain with ordered core and likelihood modules
- numeric likelihood value returned by computeLikelihood(ctx)
- chain context (ctx) carrying core-module predictions to likelihood modules

## How to apply

Define the sampled parameter array in numpy with columns [start centre, min, max, start width] (e.g. H0 = [70, 40, 100, 3]). Instantiate LikelihoodComputationChain with min=params[:,1] and max=params[:,2] so that out-of-bound parameter vectors are rejected during sampling. Store core and likelihood modules in ordered registries exposed through addCoreModule() and addLikelihoodModule(), and create a chain context (ctx) object that carries data between modules during evaluation. Register the theory code (e.g. CambCoreModule) as a core module and the data likelihood (e.g. WmapLikelihoodModule) as a likelihood module, then call chain.setup() so both are initialised before evaluation; during each likelihood evaluation the core module writes its prediction into the shared ctx and the likelihood module reads it via computeLikelihood(ctx). The rationale is that this plug-in architecture makes the framework easily extendable, since further likelihood functions or theory codes can be added as Python modules without touching the sampler.

## Related tools

- **CosmoHammer** (Framework providing LikelihoodComputationChain, addCoreModule()/addLikelihoodModule() registries, chain.setup(), and the shared ctx for module data exchange) — https://github.com/cosmo-ethz/CosmoHammer
- **CAMB** (Boltzmann integrator wrapped as CambCoreModule; writes theory predictions into the chain context for downstream likelihood modules)
- **cosmoHammerPlugins** (Collection of public core and likelihood modules (PyCamb, WMAP, Planck) that plug into the chain) — http://github.com/cosmo-ethz/CosmoHammerPlugins

## Examples

```
chain = LikelihoodComputationChain(min=params[:,1], max=params[:,2]); chain.addCoreModule(CambCoreModule()); chain.addLikelihoodModule(WmapLikelihoodModule()); chain.setup(); lnL = chain.computeLikelihood(ctx)
```

## Evaluation signals

- The script runs end-to-end without parameter-bound violations (chain constructed with min=params[:,1], max=params[:,2])
- chain.setup() successfully initialises both the CAMB core module and the WMAP likelihood module before evaluation
- computeLikelihood(ctx) returns a numeric likelihood value (not None or an error)
- The core module's prediction is visible to the likelihood module through the shared ctx within a single chain evaluation
- Additional likelihood/theory modules can be registered via addCoreModule()/addLikelihoodModule() without modifying the sampler code

## Limitations

- WMAP likelihood requires the WMAP data to be accessible on the filesystem and CFITSIO to be installed — an external dependency not bundled
- Public modules (PyCamb, WMAP, Planck) live in a separate cosmoHammerPlugins project and must be installed separately, each module individually
- Performance scales with MCMC overheads and parallelisation costs; processes idle once more processes are available than walkers

## Evidence

- [discussion] chain=Likelihood Computation Chain (min=params [: ,1] , max=params [: ,2]): [evidence span withheld — anchor status: not_found]
- [intro] The architecture of our code makes Cosmo Hammer easily extendable to other applications, as it is straight forward to plug in the Python modules containing further likelihood functions or codes for: [evidence span withheld — anchor status: not_found]
