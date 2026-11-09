---
name: wmap-likelihood-chain-configuration
description: Use when setting up cosmological parameter estimation with CosmoHammer
  against WMAP (3/5/7/9-year) CMB data, i.e. when a LikelihoodComputationChain must
  be assembled with CAMB and WMAP likelihood modules before sampling.
when_to_use_negative:
- The analysis targets likelihood codes or data other than the plug-in modules available
  in CosmoHammerPlugins (PyCamb, WMAP, Planck) without writing a new module.
- Only the default Metropolis–Hastings CosmoMC sampler is required — this skill configures
  the CosmoHammer/emcee Goodman–Weare chain, not CosmoMC.
- The WMAP data or CFITSIO are not available locally; chain.setup() cannot initialise
  the WMAP modules without them.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: CosmoHammer
  role: Provides the LikelihoodComputationChain and CosmoHammerSampler classes used
    to assemble and run the WMAP likelihood configuration
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: CosmoHammerPlugins
  role: Hosts the separately-installed PyCamb, WMAP and Planck core/likelihood modules
    plugged into the chain
  repo: http://github.com/cosmo-ethz/CosmoHammerPlugins
- name: CAMB
  role: Boltzmann integrator wrapped by CambCoreModule to supply theory predictions
    to the likelihood chain
- name: emcee
  role: Affine-invariant Goodman-Weare ensemble sampler embedded by CosmoHammer that
    consumes the configured chain during startSampling()
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:7cdc825314db7488bdf7cd8f2ecdf6fa62c727fc7600ea6b0fea9bf5e131341b
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
  - WMAP
  - acor
  - emcee
  - mpi4py
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# wmap-likelihood-chain-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure a CosmoHammer LikelihoodComputationChain that combines the CAMB Boltzmann solver core module with the WMAP 7-year likelihood modules so that MCMC sampling of the seven Lambda-CDM parameters can be launched. This is the setup layer that wires parameter bounds, start widths, and likelihood modules together before CosmoHammerSampler's startSampling() is called.

## When to use

Use when setting up cosmological parameter estimation with CosmoHammer against WMAP (3/5/7/9-year) CMB data, i.e. when a LikelihoodComputationChain must be assembled with CAMB and WMAP likelihood modules before sampling.

## When NOT to use

- The analysis targets likelihood codes or data other than the plug-in modules available in CosmoHammerPlugins (PyCamb, WMAP, Planck) without writing a new module.
- Only the default Metropolis–Hastings CosmoMC sampler is required — this skill configures the CosmoHammer/emcee Goodman–Weare chain, not CosmoMC.
- The WMAP data or CFITSIO are not available locally; chain.setup() cannot initialise the WMAP modules without them.

## Inputs

- params array of [start centre, min, max, start width] rows for the Lambda-CDM parameters (e.g. [[70, 40, 100, 3], ...])
- WMAP likelihood data files accessible on the local filesystem (e.g. WMAP 7-year likelihood data)
- CosmoHammerPlugins modules: PyCamb/CambCoreModule and WMAP likelihood modules
- CFITSIO library installed on the system

## Outputs

- Initialised LikelihoodComputationChain containing CambCoreModule and WMAP likelihood modules
- Posterior sample chains from CosmoHammerSampler.startSampling() (e.g. 350 walkers = 50 × 7 parameters, targeting ~87,500 converged samples)

## How to apply

Define a params array of parameter start centre, min, max and start width rows (e.g. [[70, 40, 100, 3], ...]) for the sampled cosmological parameters, then instantiate LikelihoodComputationChain with min=params[:,1] and max=params[:,2]. Add CambCoreModule via chain.addCoreModule(camb) and the appropriate WMAP likelihood module (e.g. WmapLikelihoodModule for the 7-year data) via chain.addLikelihoodModule(...). Call chain.setup() to initialise the CAMB and WMAP modules — the WMAP data must be accessible on the filesystem and CFITSIO installed. Launch sampling by constructing CosmoHammerSampler with the params, chain, file prefix, walkers ratio (e.g. 50, yielding 50 × n_params walkers), burn-in and sample iterations, then calling startSampling(); use the CosmoHammerPlugins project for the PyCamb, WMAP and Planck modules, each installed separately.

## Related tools

- **CosmoHammer** (Provides the LikelihoodComputationChain and CosmoHammerSampler classes used to assemble and run the WMAP likelihood configuration) — http://github.com/cosmo-ethz/CosmoHammer
- **CosmoHammerPlugins** (Hosts the separately-installed PyCamb, WMAP and Planck core/likelihood modules plugged into the chain) — http://github.com/cosmo-ethz/CosmoHammerPlugins
- **CAMB** (Boltzmann integrator wrapped by CambCoreModule to supply theory predictions to the likelihood chain)
- **emcee** (Affine-invariant Goodman-Weare ensemble sampler embedded by CosmoHammer that consumes the configured chain during startSampling())

## Examples

```
chain = LikelihoodComputationChain(min=params[:,1], max=params[:,2]); chain.addCoreModule(CambCoreModule()); chain.addLikelihoodModule(wmap7.WmapLikelihoodModule()); chain.setup(); sampler.startSampling()
```

## Evaluation signals

- chain.setup() completes without error, confirming CAMB and WMAP modules were initialised against the local WMAP data.
- The walkers ratio of 50 yields 50 × n_params walkers (e.g. 300 walkers for 6 parameters, 350 for the 7 WMAP 7-year parameters), matching the article's configuration.
- The chain's min/max boundaries equal params[:,1] and params[:,2], so sampled points stay within the parameter bounds.
- Per-parameter autocorrelation times from the resulting chain fall in the reported ranges (tau ≈ 43.4–48.2 for CosmoHammer vs 13.8–18.4 for fine-tuned CosmoMC on WMAP 7).
- Posterior means and marginal standard deviations for the seven Lambda-CDM parameters agree with CosmoMC reference estimates within the expected ~3.4%-of-sigma statistical error.

## Limitations

- The WMAP data must be accessible on the filesystem and CFITSIO has to be installed; neither is bundled with the distribution.
- Each CosmoHammerPlugins module (PyCamb, WMAP, Planck) must be installed separately; missing one silently breaks the chain.
- The current distribution has only been tested with Intel's ifort compiler and mkl libraries for the Fortran components.
- Processes idle once there are more processes than walkers, so core count should stay within the walker budget (plateau observed at 64 nodes).

## Evidence

- [discussion] Instantiate the chain with parameter bounds: [evidence span withheld — anchor status: not_found]
- [discussion] External dependencies for WMAP modules: [evidence span withheld — anchor status: not_found]
