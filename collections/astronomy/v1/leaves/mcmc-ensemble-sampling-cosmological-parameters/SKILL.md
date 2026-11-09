---
name: mcmc-ensemble-sampling-cosmological-parameters
description: Use when you need to estimate cosmological parameters (e.g. the seven
  Lambda-CDM parameters) from CMB data via MCMC, or benchmark a new sampler's posteriors
  against a CosmoMC reference chain.
when_to_use_negative:
- The goal is a single best-fit point rather than a full posterior — the paper instead
  uses particle swarm optimization before MCMC for that purpose
- Only a small number of cores is available and wall time is dominated by per-step
  likelihood cost rather than parallelisable ensemble evaluation (Metropolis-Hastings
  with heavy overheads may not benefit)
- More MPI processes are launched than there are ensemble walkers — extra processes
  idle and scaling plateaus (observed at 64 nodes)
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0102
tools:
- name: CosmoHammer
  role: Python MCMC framework embedding emcee that orchestrates the likelihood chain
    and parallel sampling
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Affine invariant ensemble sampler by Goodman and Weare (2010) used for the
    parameter inference
- name: CAMB
  role: Boltzmann integrator wrapped by the CambCoreModule to compute theory predictions
    during sampling
- name: CosmoHammerPlugins
  role: Public likelihood and core modules (PyCamb, WMAP, Planck) installed separately
    to supply the WMAP 7-year likelihood
  repo: http://github.com/cosmo-ethz/CosmoHammerPlugins
- name: acor
  role: Python wrapper estimating the autocorrelation time of the MCMC chains to assess
    convergence
  repo: https://github.com/dfm/acor
- name: CosmoMC
  role: Fortran Metropolis-Hastings reference sampler whose published parameter estimates
    benchmark the CosmoHammer posteriors
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:d07f228eac14a40f2e8fc0b6de71be57ff1bae7254faf1e06a76276a4f860e84
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
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# mcmc-ensemble-sampling-cosmological-parameters

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run parallelised affine-invariant ensemble MCMC (emcee via CosmoHammer) over Lambda-CDM parameters with CAMB and the WMAP 7-year likelihood to obtain converged posterior means and marginal standard deviations comparable to CosmoMC within a ~3.4%-of-sigma statistical error.

## When to use

Use when you need to estimate cosmological parameters (e.g. the seven Lambda-CDM parameters) from CMB data via MCMC, or benchmark a new sampler's posteriors against a CosmoMC reference chain.

## When NOT to use

- The goal is a single best-fit point rather than a full posterior — the paper instead uses particle swarm optimization before MCMC for that purpose
- Only a small number of cores is available and wall time is dominated by per-step likelihood cost rather than parallelisable ensemble evaluation (Metropolis-Hastings with heavy overheads may not benefit)
- More MPI processes are launched than there are ensemble walkers — extra processes idle and scaling plateaus (observed at 64 nodes)

## Inputs

- params array with start centre, min, max and start width for the seven Lambda-CDM parameters (e.g. [70, 40, 100, 3] for H0)
- WMAP 7-year likelihood data accessible on the local filesystem
- CAMB Boltzmann solver code (Fortran90, Intel ifort/mkl tested)
- CosmoMC reference chain or its published posterior summary values (N = 35,000 samples)

## Outputs

- posterior_summary.csv with mean and marginal standard deviation columns for all seven Lambda-CDM parameters
- Retained MCMC posterior samples (target 87,500 converged samples after burn-in)
- Estimated autocorrelation times per parameter from the acor module

## How to apply

Install CosmoHammer and the CosmoHammerPlugins modules (PyCamb, WMAP) into a Python environment where CAMB and the WMAP 7-year likelihood data are accessible on the filesystem (CFITSIO installed). Define a params array of parameter start centre, min, max and start width (e.g. H0 start centre 70, min 40, max 100, width 3), build a LikelihoodComputationChain with these min/max bounds, and add the CambCoreModule plus the WMAP 7-year likelihood module before calling chain.setup(). Create a CosmoHammerSampler with a walkers ratio of 50 (50 x 7 = 350 walkers for seven parameters), a burn-in phase, and enough sample iterations to accumulate 87,500 converged posterior samples, then call startSampling(); optionally use mpi4py or run on Amazon EC2 to scale out, noting the plateau once processes exceed walkers. Discard burn-in and compute the posterior mean and marginal standard deviation per parameter, and verify agreement with the CosmoMC reference (N = 35,000 samples) within the expected ~3.4%-of-sigma statistical error scale. Optionally estimate autocorrelation times with the acor module to confirm convergence.

## Related tools

- **CosmoHammer** (Python MCMC framework embedding emcee that orchestrates the likelihood chain and parallel sampling) — http://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Affine invariant ensemble sampler by Goodman and Weare (2010) used for the parameter inference)
- **CAMB** (Boltzmann integrator wrapped by the CambCoreModule to compute theory predictions during sampling)
- **CosmoHammerPlugins** (Public likelihood and core modules (PyCamb, WMAP, Planck) installed separately to supply the WMAP 7-year likelihood) — http://github.com/cosmo-ethz/CosmoHammerPlugins
- **acor** (Python wrapper estimating the autocorrelation time of the MCMC chains to assess convergence) — https://github.com/dfm/acor
- **CosmoMC** (Fortran Metropolis-Hastings reference sampler whose published parameter estimates benchmark the CosmoHammer posteriors)

## Examples

```
chain = LikelihoodComputationChain(min=params[:,1], max=params[:,2]); chain.addCoreModule(camb); chain.addLikelihoodModule(wmap7.WmapLikelihoodModule()); sampler.startSampling()
```

## Evaluation signals

- File posterior_summary.csv exists and contains mean and marginal standard deviation columns for all seven Lambda-CDM parameters
- Each parameter estimate deviates from the Table 3 CosmoMC reference by no more than the reported ~3.4%-of-sigma tolerance
- Parameter estimates remain within the expected 3.4%-of-sigma statistical error scale given the chosen sample sizes (87,500 vs 35,000)
- Walkers count equals walkers ratio times number of parameters (50 x 7 = 350 walkers for Lambda-CDM)
- Autocorrelation times from acor indicate chains have converged before posterior summary extraction

## Limitations

- WMAP data must be accessible on the local filesystem and CFITSIO must be installed — it is not bundled with the distribution
- Distribution only tested with Intel's ifort compiler and mkl libraries; other Fortran compilers are untested
- Scaling saturates once there are more processes than walkers (plateau at 64 nodes); doubling cores yields 11% scaling overhead (alpha = 0.89), not perfect linear scaling
- mpi4py support is optional and only used for multi-node distribution
- Software tarballs were hosted on an ETH Zurich personal page rather than a versioned archive or DOI; no changelog was found in the repository

## Evidence

- [discussion] Chain instantiation with min/max boundaries: [evidence span withheld — anchor status: not_found]
- [other] Finding: estimates agree within expected statistical error scale: [evidence span withheld — anchor status: not_found]
- [discussion] External data dependency on local filesystem: [evidence span withheld — anchor status: not_found]
