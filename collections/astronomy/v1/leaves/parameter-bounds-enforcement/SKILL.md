---
name: parameter-bounds-enforcement
description: Use when assembling a CosmoHammer sampling configuration from core/likelihood
  modules (e.g. CambCoreModule + WMAP) and sampled parameters such as H0 must remain
  within physically motivated min/max bounds during the chain.
when_to_use_negative:
- When parameters are unconstrained or the sampler handles bounds itself (e.g. during
  particle swarm optimization via ParticleSwarmOptimizer with params[:,1], params[:,2],
  which takes bounds as its own arguments)
- When the goal is likelihood-only evaluation of a fixed best-fit point with no sampling
  of new parameter proposals
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: CosmoHammer
  role: Provides LikelihoodComputationChain, which stores modules in order, initialises
    them via setup(), and rejects sampled parameters outside the min/max bounds
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: CAMB
  role: Boltzmann integrator wrapped as CambCoreModule; its predictions are written
    into the chain context only for in-bound parameter points
- name: emcee
  role: Affine invariant ensemble sampler embedded by CosmoHammer; its walkers generate
    proposals that the chain screens against the bounds
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:485fa0b1a99279d7a112b6323a5cf823077acdad65fabc9f61304e35c0442608
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
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# parameter-bounds-enforcement

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Enforces physically motivated parameter bounds during MCMC sampling by instantiating CosmoHammer's LikelihoodComputationChain with min/max arrays derived from the sampling parameter table, so out-of-bound proposals are rejected before likelihood evaluation.

## When to use

Use when assembling a CosmoHammer sampling configuration from core/likelihood modules (e.g. CambCoreModule + WMAP) and sampled parameters such as H0 must remain within physically motivated min/max bounds during the chain.

## When NOT to use

- When parameters are unconstrained or the sampler handles bounds itself (e.g. during particle swarm optimization via ParticleSwarmOptimizer with params[:,1], params[:,2], which takes bounds as its own arguments)
- When the goal is likelihood-only evaluation of a fixed best-fit point with no sampling of new parameter proposals

## Inputs

- numpy parameter array with columns [start centre, min, max, start width], e.g. params = np.array([[70, 40, 100, 3], ...])
- Core module instance (e.g. CambCoreModule) and likelihood module instance (e.g. WmapLikelihoodModule)
- Chain context (ctx) object carrying data between modules during evaluation

## Outputs

- LikelihoodComputationChain configured with min=params[:,1] and max=params[:,2]
- Numeric likelihood value returned by computeLikelihood(ctx) with no parameter-bound violations

## How to apply

First define the sampling parameter array in numpy with columns [start centre, min, max, start width], e.g. H0 = [70, 40, 100, 3]. Instantiate LikelihoodComputationChain with min=params[:,1] and max=params[:,2] so that any proposal outside the physical range is rejected rather than passed to the modules. Register core modules (e.g. CambCoreModule via chain.addCoreModule) and likelihood modules (e.g. wmap.WmapLikelihoodModule via chain.addLikelihoodModule); the chain stores them in order and, during evaluation, the core module writes its prediction into the shared chain context (ctx) which the likelihood module reads via computeLikelihood(ctx). Call chain.setup() so the CAMB and WMAP modules are initialised before sampling. The same min/max arrays are reused when constructing the CosmoHammerSampler (walkers ratio 50 launches 50 x 6 = 300 walkers for six parameters), keeping bounds consistent between chain rejection and sampler initialisation.

## Related tools

- **CosmoHammer** (Provides LikelihoodComputationChain, which stores modules in order, initialises them via setup(), and rejects sampled parameters outside the min/max bounds) — http://github.com/cosmo-ethz/CosmoHammer
- **CAMB** (Boltzmann integrator wrapped as CambCoreModule; its predictions are written into the chain context only for in-bound parameter points)
- **emcee** (Affine invariant ensemble sampler embedded by CosmoHammer; its walkers generate proposals that the chain screens against the bounds)

## Examples

```
params = np.array([[70, 40, 100, 3]]); chain = LikelihoodComputationChain(min=params[:,1], max=params[:,2]); chain.addCoreModule(CambCoreModule()); chain.addLikelihoodModule(wmap.WmapLikelihoodModule()); chain.setup()
```

## Evaluation signals

- chain.setup() initialises both the CAMB and WMAP modules without error
- computeLikelihood(ctx) returns a numeric likelihood value
- No parameter-bound violations occur during sampling (all accepted points satisfy min=params[:,1], max=params[:,2])
- The script runs end-to-end from parameter array definition through chain setup to likelihood computation

## Limitations

- Bounds only protect the likelihood evaluation inside the chain; separate bounds must be passed to ParticleSwarmOptimizer for the PSO pre-fit stage
- Scaling saturates when more processes are available than walkers, causing processes and their spawned threads to idle

## Evidence

- [other] The Likelihood Computation Chain control structure stores the modules and initialises them in the right order, and it ensures that sampled parameters remain within physically motivated bounds during the sampling process.: [evidence span withheld — anchor status: not_found]
- [discussion] Instantiate LikelihoodComputationChain with min/max boundaries: [evidence span withheld — anchor status: not_found]
- [intro] The architecture of our code makes Cosmo Hammer easily extendable to other applications, as it is straight forward to plug in the Python modules containing further likelihood functions or codes: [evidence span withheld — anchor status: not_found]
