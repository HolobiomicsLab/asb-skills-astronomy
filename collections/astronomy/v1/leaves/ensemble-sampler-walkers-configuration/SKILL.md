---
name: ensemble-sampler-walkers-configuration
description: Use when running MCMC sampling of CMB/WMAP cosmological parameters with
  CosmoHammer and you must size the emcee walker ensemble and configure the LikelihoodComputationChain
  before sampling.
when_to_use_negative:
- The analysis does not use an ensemble (Goodman–Weare) sampler — e.g. you are running
  CosmoMC's Metropolis–Hastings sampler, where no walker ratio is defined
- You have more MPI processes than walkers, so processes would idle and scaling saturates
  instead of improving wall time
- Only a wall-clock scaling benchmark is needed without parameter inference — the
  skill concerns sampler configuration, not core-count scaling analysis
tools:
- name: CosmoHammer
  role: Python framework that wraps the emcee ensemble sampler; its CosmoHammerSampler
    is configured with walkersRatio, burn-in and sample iterations
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Implementation of the Goodman–Weare affine invariant ensemble sampler that
    is embedded and driven by CosmoHammer
- name: CosmoHammerPlugins
  role: Provides the CambCoreModule and WMAP likelihood modules that are plugged into
    the LikelihoodComputationChain
  repo: http://github.com/cosmo-ethz/CosmoHammerPlugins
- name: acor
  role: Python port of Goodman's ACOR routine used to estimate the autocorrelation
    time tau of the sampled chains for validation
  repo: https://github.com/dfm/acor
- name: mpi4py
  role: Optional wrapper used to distribute CosmoHammer sampling across multiple nodes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:5367d57ba0ba7ce0f2b9b9f6310ec517235368a46dd88b3a0c81d51bb1157e23
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# ensemble-sampler-walkers-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure the Goodman–Weare affine-invariant ensemble sampler embedded in CosmoHammer (emcee) for cosmological parameter estimation, setting the walkers ratio relative to the number of sampled parameters so the ensemble is large enough for reliable autocorrelation-time estimates.

## When to use

Use when running MCMC sampling of CMB/WMAP cosmological parameters with CosmoHammer and you must size the emcee walker ensemble and configure the LikelihoodComputationChain before sampling.

## When NOT to use

- The analysis does not use an ensemble (Goodman–Weare) sampler — e.g. you are running CosmoMC's Metropolis–Hastings sampler, where no walker ratio is defined
- You have more MPI processes than walkers, so processes would idle and scaling saturates instead of improving wall time
- Only a wall-clock scaling benchmark is needed without parameter inference — the skill concerns sampler configuration, not core-count scaling analysis

## Inputs

- Parameter configuration array with start centre, min, max, and start width per sampled cosmological parameter (e.g. H0, Omega_b, Omega_cdm, tau, n_s, logA, sigma8)
- Likelihood modules from cosmoHammerPlugins (e.g. WmapLikelihoodModule for WMAP 7-year data) and CambCoreModule
- Locally accessible WMAP likelihood data on the filesystem (with CFITSIO installed)
- MCMC chain output time series per parameter for autocorrelation diagnostics

## Outputs

- Ensemble of MCMC samples from emcee walkers (e.g. 350 walkers for 7 parameters at walkersRatio=50)
- Burn-in and sampled chain files written under the chosen file prefix
- Per-parameter estimated autocorrelation times tau (via acor.acor returning tau, mean, sigma)

## How to apply

Define a parameter array with start centre, min, max and start width per parameter (e.g. params = np.array([[70, 40, 100, 3], ...]) for WMAP 7), then instantiate a LikelihoodComputationChain with min=params[:,1] and max=params[:,2] and add the CambCoreModule and WMAP likelihood modules before calling chain.setup(). Size the ensemble with a walkers ratio of 50, so that the total number of emcee walkers equals 50 × (number of sampled parameters) — e.g. 50 × 6 = 300 walkers for six parameters, or 350 walkers for the seven WMAP 7 parameters. Create a CosmoHammerSampler with the params, chain, a file prefix, the walkers ratio, burn-in iterations and sample iterations, then call startSampling(); use mpi4py only if distributing across multiple nodes. Because scaling saturates once more processes than walkers exist, keep the walker count above the core count to avoid idling. After sampling, validate convergence by computing the exponential autocorrelation time per parameter with the Goodman–Weare Appendix C estimator via the acor package (pip install acor), where CosmoHammer configurations with ~50× walkers gave tau ≈ 43–48 versus ~14–19 for fine-tuned CosmoMC.

## Related tools

- **CosmoHammer** (Python framework that wraps the emcee ensemble sampler; its CosmoHammerSampler is configured with walkersRatio, burn-in and sample iterations) — http://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Implementation of the Goodman–Weare affine invariant ensemble sampler that is embedded and driven by CosmoHammer)
- **CosmoHammerPlugins** (Provides the CambCoreModule and WMAP likelihood modules that are plugged into the LikelihoodComputationChain) — http://github.com/cosmo-ethz/CosmoHammerPlugins
- **acor** (Python port of Goodman's ACOR routine used to estimate the autocorrelation time tau of the sampled chains for validation) — https://github.com/dfm/acor
- **mpi4py** (Optional wrapper used to distribute CosmoHammer sampling across multiple nodes)

## Examples

```
chain = LikelihoodComputationChain(min=params[:,1], max=params[:,2]); chain.addCoreModule(CambCoreModule()); chain.addLikelihoodModule(wmap7.WmapLikelihoodModule()); chain.setup(); CosmoHammerSampler(params=params, chain=chain, filePrefix='wmap7_', walkersRatio=50, burninIterations=500, sampleIterations=1000).startSampling()
```

## Evaluation signals

- Walker count equals walkersRatio × number of sampled parameters (e.g. 50 × 7 = 350 walkers for the WMAP 7 parameter space, as in the paper)
- The chain runs through chain.setup() and startSampling() without errors and writes chain files under the given file prefix
- Reproduced per-parameter autocorrelation times tau fall in the ranges reported in Table 2: CosmoHammer tau ≈ 43.4–48.2 and fine-tuned CosmoMC ≈ 13.8–18.4
- tau values computed with acor.acor(x) return finite tau, mean, sigma for each sampled parameter
- Core count does not exceed walker count; scaling saturates (idle processes) otherwise, indicating misconfiguration

## Limitations

- Scaling saturates once there are more processes than walkers, causing processes and their spawned threads to idle (plateau observed at 64 nodes)
-  mpi4py support is optional and only needed for multi-node distribution
- The WMAP data must be locally accessible on the filesystem and CFITSIO has to be installed; these dependencies are not bundled
- The current distribution has only been tested with Intel's ifort compiler and mkl libraries
- emcee/Goodman–Weare sampling via CosmoHammer yields larger autocorrelation times (tau ≈ 44) than fine-tuned Metropolis–Hastings CosmoMC (tau ≈ 15–18) on the same WMAP 7 likelihood

## Evidence

- [discussion] the WMAP data needs to be accessible on the filesystem and CFITSIO has to be installed.: [evidence span withheld — anchor status: not_found]
