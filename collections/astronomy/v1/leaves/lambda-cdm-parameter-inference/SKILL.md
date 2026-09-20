---
name: lambda-cdm-parameter-inference
description: Use when a task requires MCMC-based posterior estimation of the seven
  Lambda-CDM parameters from WMAP likelihood data, or benchmarking an emcee-based
  sampler against CosmoMC.
when_to_use_negative:
- The goal is wall-time optimisation or scaling analysis of the sampler itself rather
  than posterior parameter estimates (use a scaling benchmark skill instead)
- Only a single likelihood evaluation at a fixed parameter point is needed — CosmoHammer
  is built for full posterior sampling, and Metropolis-Hastings is dominated by overheads
  in parallelised runs
- No CAMB / WMAP likelihood data can be placed on the filesystem, since the modules
  cannot be initialised without them
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: CosmoHammer
  role: Python framework that embeds emcee and orchestrates the parallelised MCMC
    sampling of the Lambda-CDM posterior
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Affine invariant ensemble sampler (Goodman and Weare algorithm) driving the
    walker moves during sampling
- name: CAMB
  role: Boltzmann integrator computing theory predictions inside the CambCoreModule
    of the LikelihoodComputationChain
- name: CosmoHammerPlugins
  role: Provides the PyCamb, WMAP and Planck likelihood/core modules that must be
    installed separately
  repo: http://github.com/cosmo-ethz/CosmoHammerPlugins
- name: CosmoMC
  role: Reference Metropolis-Hastings sampler whose published posterior estimates
    serve as the benchmark (Table 3)
- name: acor
  role: Python wrapper estimating the autocorrelation time of the sampled chains
  repo: https://github.com/dfm/acor
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:4011038cf0f9feafc292dcb1f1bc67a1bad017a9abe98fad553c5337b439ecd1
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
  - CosmoMC
  - Python
  - acor
  - emcee
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lambda-cdm-parameter-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers the seven Lambda-CDM cosmological parameters from WMAP CMB data by sampling the posterior with CosmoHammer's parallelised emcee ensemble sampler over CAMB theory predictions, and validates agreement with CosmoMC reference estimates.

## When to use

Use when a task requires MCMC-based posterior estimation of the seven Lambda-CDM parameters from WMAP likelihood data, or benchmarking an emcee-based sampler against CosmoMC.

## When NOT to use

- The goal is wall-time optimisation or scaling analysis of the sampler itself rather than posterior parameter estimates (use a scaling benchmark skill instead)
- Only a single likelihood evaluation at a fixed parameter point is needed — CosmoHammer is built for full posterior sampling, and Metropolis-Hastings is dominated by overheads in parallelised runs
- No CAMB / WMAP likelihood data can be placed on the filesystem, since the modules cannot be initialised without them

## Inputs

- Seven Lambda-CDM parameter start centres, min/max boundaries and start widths as a params array (e.g. params = np.array([[70, 40, 100, 3], ...]))
- WMAP 7-year likelihood data accessible on the filesystem
- CosmoMC reference chain (N = 35,000 samples) or its published posterior summary (Table 3)

## Outputs

- posterior_summary.csv with a mean and marginal standard deviation column for all seven Lambda-CDM parameters
- Per-parameter deviation from the Table 3 CosmoMC reference, quantified against the ~3.4%-of-sigma tolerance

## How to apply

Install CosmoHammer and its WMAP/PyCamb plugin modules from the cosmo-ethz GitHub repositories, ensuring CAMB, the WMAP likelihood data on the filesystem, and CFITSIO are available. Define a params array holding the seven Lambda-CDM parameter start centres, min/max boundaries, and start widths (e.g. rows like [70, 40, 100, 3]), build a LikelihoodComputationChain with those min/max bounds, add CambCoreModule plus the WMAP 7-year likelihood module, and call chain.setup() to initialise them. Launch CosmoHammerSampler with a walkers ratio of 50 (yielding 350 walkers for 7 parameters), a burn-in phase, and enough sample iterations to accumulate ~87,500 converged posterior samples via startSampling(). After discarding burn-in, compute the posterior mean and marginal standard deviation per parameter, then verify against the CosmoMC reference (N = 35,000 samples): estimates must agree within the expected statistical error of about 3.4% of the standard deviation. Monitor convergence with the acor autocorrelation-time estimator; beware that scaling saturates once processes outnumber walkers.

## Related tools

- **CosmoHammer** (Python framework that embeds emcee and orchestrates the parallelised MCMC sampling of the Lambda-CDM posterior) — http://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Affine invariant ensemble sampler (Goodman and Weare algorithm) driving the walker moves during sampling)
- **CAMB** (Boltzmann integrator computing theory predictions inside the CambCoreModule of the LikelihoodComputationChain)
- **CosmoHammerPlugins** (Provides the PyCamb, WMAP and Planck likelihood/core modules that must be installed separately) — http://github.com/cosmo-ethz/CosmoHammerPlugins
- **CosmoMC** (Reference Metropolis-Hastings sampler whose published posterior estimates serve as the benchmark (Table 3))
- **acor** (Python wrapper estimating the autocorrelation time of the sampled chains) — https://github.com/dfm/acor

## Examples

```
chain = LikelihoodComputationChain(min=params[:,1], max=params[:,2]); chain.addCoreModule(CambCoreModule()); chain.setup(); sampler.startSampling()
```

## Evaluation signals

- File posterior_summary.csv exists and contains mean and marginal standard deviation columns for all seven Lambda-CDM parameters
- Each parameter estimate deviates from the Table 3 CosmoMC reference by no more than the ~3.4%-of-sigma expected statistical error
- Walker count equals walkers ratio × number of parameters (50 × 7 = 350 walkers; the paper's 6-parameter example gives 300)
- Burn-in samples are discarded before computing posterior summaries
- Convergence is supported by a finite acor autocorrelation time for the retained chains

## Limitations

- Scaling saturates: once more processes are available than walkers, processes and their spawned threads idle (plateau observed at 64 nodes)
- Doubling cores yields an 11% scaling overhead (α = 0.89), not perfectly linear wall-time scaling
- WMAP data must be locally accessible on the filesystem and CFITSIO has to be installed; the data are not bundled
- Only tested with Intel's ifort compiler and mkl libraries; other Fortran compilers are untested
- mpi4py is optional and only needed when distributing CosmoHammer over multiple nodes

## Evidence

- [discussion] chain construction: [evidence span withheld — anchor status: not_found]
- [other] agreement tolerance vs CosmoMC: [evidence span withheld — anchor status: not_found]
- [discussion] WMAP data dependency: [evidence span withheld — anchor status: not_found]
