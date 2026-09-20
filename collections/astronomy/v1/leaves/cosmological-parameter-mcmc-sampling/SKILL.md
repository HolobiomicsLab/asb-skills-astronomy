---
name: cosmological-parameter-mcmc-sampling
description: Use when a task requires setting up or benchmarking MCMC posterior sampling
  of cosmological parameters (H0, Omega_b, Omega_cdm, tau, n_s, logA, sigma8) from
  CMB likelihoods, or when reproducing autocorrelation times, burn-in lengths, or
  epsilon-convergence comparisons between ensemble and.
when_to_use_negative:
- The analysis targets likelihood-code benchmarks or scalability (wall-time vs number
  of cores) rather than posterior sampling quality and autocorrelation diagnostics.
- Only a single-chain Metropolis–Hastings sampler without ensemble walkers is available,
  so Goodman–Weare autocorrelation estimators and walkers-ratio configuration do not
  apply.
- The posterior is for a non-CMB likelihood without a CAMB/core module or WMAP/Planck
  likelihood plugin available in CosmoHammerPlugins.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: CosmoHammer
  role: Python framework that embeds emcee and orchestrates the LikelihoodComputationChain,
    walker configuration, burn-in, and sampling for WMAP/CAMB likelihoods
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Implementation of the Goodman–Weare affine-invariant ensemble sampler used
    for the parallelised posterior sampling
- name: CAMB
  role: Boltzmann integrator added as the CambCoreModule to compute theory predictions
    inside the likelihood chain
- name: CosmoMC
  role: Metropolis–Hastings baseline (default, fine-tuned, and self-tuning configurations)
    against which CosmoHammer's tau, burn-in, and epsilon are compared
- name: acor
  role: Python wrapper of Goodman's ACOR routine used to estimate the exponential
    autocorrelation time tau per parameter from the chains
  repo: https://github.com/dfm/acor
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:91ac9d904ba0999a2937d69f782dcc82176bf83fdd56b5a74c3c81dc09b7179d
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
  - CosmoMC
  - Python
  - acor
  - emcee
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cosmological-parameter-mcmc-sampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate cosmological parameters from CMB data (e.g. WMAP 7-year likelihood) by running the emcee Goodman–Weare affine-invariant ensemble sampler via CosmoHammer against the CAMB Boltzmann integrator, and compare sampling efficiency with Metropolis–Hastings CosmoMC runs using autocorrelation-time diagnostics.

## When to use

Use when a task requires setting up or benchmarking MCMC posterior sampling of cosmological parameters (H0, Omega_b, Omega_cdm, tau, n_s, logA, sigma8) from CMB likelihoods, or when reproducing autocorrelation times, burn-in lengths, or epsilon-convergence comparisons between ensemble and Metropolis–Hastings samplers.

## When NOT to use

- The analysis targets likelihood-code benchmarks or scalability (wall-time vs number of cores) rather than posterior sampling quality and autocorrelation diagnostics.
- Only a single-chain Metropolis–Hastings sampler without ensemble walkers is available, so Goodman–Weare autocorrelation estimators and walkers-ratio configuration do not apply.
- The posterior is for a non-CMB likelihood without a CAMB/core module or WMAP/Planck likelihood plugin available in CosmoHammerPlugins.

## Inputs

- Parameter grid as numpy array of rows [start centre, min, max, start width], e.g. [[70, 40, 100, 3], ...]
- WMAP 7-year likelihood data accessible on the local filesystem (with CFITSIO installed)
- CAMB Boltzmann integrator and WMAP likelihood wrapper modules from the CosmoHammerPlugins project
- Posterior sample chains from CosmoHammer (emcee) and CosmoMC runs

## Outputs

- Per-parameter autocorrelation times tau (expected ≈ 43.4–48.2 for CosmoHammer, 13.8–18.4 for fine-tuned CosmoMC, 13.3–19.1 for self-tuning CosmoMC)
- Burn-in length determination (250 steps for CosmoHammer and fine-tuned CosmoMC; ~2000-step tuning phase for self-tuning CosmoMC)
- epsilon-convergence figure (e.g. epsilon_convergence_curves.png) and burn-in trajectory figure with sigma/sqrt(L) band
- Posterior sample files from CosmoHammer's CosmoHammerSampler

## How to apply

Configure a LikelihoodComputationChain with the CAMB core module and the WMAP 7-year likelihood module, supplying parameter start/min/max/start-width arrays (e.g. [[70, 40, 100, 3], ...]). Launch the emcee sampler with a walkers ratio of 50, which yields 50 × N_params walkers (300 for six parameters, 350 for seven for WMAP 7), and use a burn-in of 250 sequential steps, the value at which the ensemble mean X̄_t fluctuates within the predicted standard-error band sigma/sqrt(L). For diagnostics, compute the exponential autocorrelation time tau per parameter with the Goodman–Weare Appendix C estimator implemented in the acor package, and evaluate the statistical error epsilon = |2*tau/((n-b)*L)|^(1/2) over grids of walkers L and sequential steps n to compare against self-tuning CosmoMC (whose proposal tuning ends at n ≈ 2000). Correct application is judged by tau values matching Table 2 (CosmoHammer ≈ 43.4–48.2 vs fine-tuned CosmoMC ≈ 13.8–18.4) and by CosmoHammer reaching epsilon ≤ 0.1 before n = 2000 even for as few as 10 walkers.

## Related tools

- **CosmoHammer** (Python framework that embeds emcee and orchestrates the LikelihoodComputationChain, walker configuration, burn-in, and sampling for WMAP/CAMB likelihoods) — http://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Implementation of the Goodman–Weare affine-invariant ensemble sampler used for the parallelised posterior sampling)
- **CAMB** (Boltzmann integrator added as the CambCoreModule to compute theory predictions inside the likelihood chain)
- **CosmoMC** (Metropolis–Hastings baseline (default, fine-tuned, and self-tuning configurations) against which CosmoHammer's tau, burn-in, and epsilon are compared)
- **acor** (Python wrapper of Goodman's ACOR routine used to estimate the exponential autocorrelation time tau per parameter from the chains) — https://github.com/dfm/acor

## Examples

```
import acor; tau, mean, sigma = acor.acor(chain_samples[:, param])
```

## Evaluation signals

- Reproduced per-parameter tau values fall in the Table 2 ranges: ≈ 43.4–48.2 for CosmoHammer and ≈ 13.8–18.4 for fine-tuned CosmoMC across the seven WMAP 7 parameters.
- The burn-in summary shows 250 sequential steps as a safe burn-in for both CosmoHammer and fine-tuned CosmoMC, and roughly 2000 tuning steps for self-tuning CosmoMC.
- After burn-in, the ensemble mean sequential step X̄_t remains within the sigma/sqrt(L) band, indicating the initialisation bias has vanished.
- The epsilon = |2*tau/((n-b)*L)|^(1/2) curve shows CosmoHammer reaching epsilon ≤ 0.1 below n = 2000, i.e. before the self-tuning CosmoMC run finalises its proposal tuning, even at L = 10 walkers.
- A walkers ratio of 50 produces 50 × N_params walkers (e.g. 350 walkers for the 7-parameter WMAP 7 setup), matching the paper's configuration.

## Limitations

- Scaling saturates: processes idle once there are more processes than walkers (plateau observed at 64 nodes), so adding cores beyond the walker count gives no benefit.
- Doubling the number of cores incurs an 11% scaling overhead (α = 0.89), so wall-time scaling is not perfectly linear.
- The WMAP data must be accessible on the filesystem and CFITSIO must be installed; these external dependencies are not bundled.
- The current distribution has only been tested with Intel's ifort compiler and mkl libraries; other Fortran compilers are untested.
- mpi4py support is optional and only needed if CosmoHammer is distributed across multiple nodes.

## Evidence

- [abstract] CosmoHammer is able to accelerate the sampling process from a wall time of 30 h on a dual core notebook to 16 min by scaling out to 2048 cores: "CosmoHammer is able to accelerate the sampling process from a wall time of 30 h on a dual core notebook to 16 min by scaling out to 2048 cores"
- [discussion] A walkers Ratio of 50 will launch 50 × 6 = 300 walkers, where six is the number of parameters sampled.: "A walkers Ratio of 50 will launch 50 × 6 = 300 walkers, where six is the number of parameters sampled."
- [discussion] the WMAP data needs to be accessible on the filesystem and CFITSIO has to be installed.: [evidence span withheld — anchor status: not_found]
- [readme] Given some time series ``x``, you can estimate the autocorrelation time (``tau``) using: ::    import acor    tau, mean, sigma = acor.acor(x): [evidence span withheld — anchor status: external_doc]
