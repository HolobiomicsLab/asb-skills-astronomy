---
name: mcmc-convergence-verification-against-reference
description: Use when an MCMC posterior sample of cosmological parameters (e.g. from
  CosmoHammer/emcee on WMAP 7 data) must be cross-checked against a reference sampler
  like CosmoMC to confirm convergence and statistical agreement within ~3.4%-of-sigma
  tolerance.
when_to_use_negative:
- The WMAP likelihood data is not accessible on the filesystem and CFITSIO is not
  installed — the likelihood modules cannot be evaluated, so no posterior can be produced
  to verify.
- The goal is throughput/scalability benchmarking of the sampler rather than posterior
  correctness — use CosmoHammer's scaling analysis (2048-core timing) instead.
- The reference posterior comes from a different dataset (e.g. Planck) or a different
  parameter basis than the seven Λ-CDM parameters, making a 3.4%-of-sigma comparison
  meaningless.
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CosmoHammer
  role: Python framework embedding emcee that launches the ensemble MCMC sampling
    whose posterior is being verified
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: emcee
  role: Affine invariant Goodman-and-Weare ensemble sampler underlying the posterior
    samples being compared
  repo: http://arxiv.org/abs/1202.3665
- name: CosmoMC
  role: Metropolis–Hastings reference chain (Fortran, CAMB-based) whose published
    parameter estimates serve as the comparison target
- name: acor
  role: Estimates the autocorrelation time tau of the sampled time series to assess
    independent-sample counts before comparison
  repo: https://github.com/dfm/acor
- name: CosmoHammerPlugins
  role: Provides the PyCamb and WMAP likelihood/core modules needed to evaluate the
    posterior during sampling
  repo: http://github.com/cosmo-ethz/CosmoHammerPlugins
- name: CAMB
  role: Boltzmann integrator used via CambCoreModule to compute theory predictions
    inside the likelihood chain
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:1adb0fe4e82ee87a33576ac31337dadf4dac012fcaac386358cc8055c29fea37
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# mcmc-convergence-verification-against-reference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Verifies that a converged MCMC posterior (e.g. CosmoHammer/emcee on the seven Λ-CDM parameters against WMAP 7-year data) agrees with a reference chain such as CosmoMC within the expected statistical error of ~3.4% of the marginal standard deviation, at matched effective sample sizes.

## When to use

Use when an MCMC posterior sample of cosmological parameters (e.g. from CosmoHammer/emcee on WMAP 7 data) must be cross-checked against a reference sampler like CosmoMC to confirm convergence and statistical agreement within ~3.4%-of-sigma tolerance.

## When NOT to use

- The WMAP likelihood data is not accessible on the filesystem and CFITSIO is not installed — the likelihood modules cannot be evaluated, so no posterior can be produced to verify.
- The goal is throughput/scalability benchmarking of the sampler rather than posterior correctness — use CosmoHammer's scaling analysis (2048-core timing) instead.
- The reference posterior comes from a different dataset (e.g. Planck) or a different parameter basis than the seven Λ-CDM parameters, making a 3.4%-of-sigma comparison meaningless.

## Inputs

- Retained (burn-in-discarded) posterior sample chain from CosmoHammer/emcee, e.g. 87,500 samples for seven Λ-CDM parameters
- Parameter array with start centre, min, max, and start width per parameter (e.g. [70, 40, 100, 3] for H0)
- CosmoMC reference chain or published Table 3 posterior means and standard deviations (N = 35,000 samples)
- WMAP 7-year likelihood data accessible on the filesystem, plus CAMB as the Boltzmann integrator

## Outputs

- posterior_summary.csv containing a mean and a marginal standard deviation column for all seven Λ-CDM parameters
- Convergence verdict: per-parameter deviation from the CosmoMC/Table 3 reference relative to sigma, expected to be within ~3.4% of sigma
- Optional autocorrelation time tau per parameter series for effective-sample-size assessment

## How to apply

Run the ensemble sampler with a walkers ratio of 50 (50 × 7 = 350 walkers for the seven Λ-CDM parameters), a burn-in phase, and enough iterations to accumulate the target sample count (e.g. 87,500 retained samples); the walkers ratio is what distinguishes the Goodman-and-Weare ensemble approach from the Metropolis–Hastings reference. Discard the burn-in phase and compute, for each parameter, the posterior mean and marginal standard deviation from the retained samples (optionally using acor to estimate the autocorrelation time tau and hence the number of independent samples). Repeat the posterior-summary extraction for the CosmoMC reference chain (N = 35,000 samples) or take its published Table 3 values. For each of the seven parameters, check that the estimate difference divided by the marginal sigma is no more than the expected statistical error of about 3.4% of sigma implied by the chosen sample size. Record the result in a posterior_summary.csv with mean and marginal standard deviation columns so the comparison is auditable.

## Related tools

- **CosmoHammer** (Python framework embedding emcee that launches the ensemble MCMC sampling whose posterior is being verified) — http://github.com/cosmo-ethz/CosmoHammer
- **emcee** (Affine invariant Goodman-and-Weare ensemble sampler underlying the posterior samples being compared) — http://arxiv.org/abs/1202.3665
- **CosmoMC** (Metropolis–Hastings reference chain (Fortran, CAMB-based) whose published parameter estimates serve as the comparison target)
- **acor** (Estimates the autocorrelation time tau of the sampled time series to assess independent-sample counts before comparison) — https://github.com/dfm/acor
- **CosmoHammerPlugins** (Provides the PyCamb and WMAP likelihood/core modules needed to evaluate the posterior during sampling) — http://github.com/cosmo-ethz/CosmoHammerPlugins
- **CAMB** (Boltzmann integrator used via CambCoreModule to compute theory predictions inside the likelihood chain)

## Examples

```
sampler = CosmoHammerSampler(params=params, chain=chain, filePrefix="wmap7_", walkersRatio=50, burnin=1000, sampleIterations=1000); sampler.startSampling()
```

## Evaluation signals

- posterior_summary.csv exists and contains a mean and a marginal standard deviation column for all seven Λ-CDM parameters
- Each parameter estimate deviates from the Table 3 / CosmoMC reference by no more than the expected ~3.4%-of-sigma statistical error for the sample size
- Burn-in samples are excluded before computing means and marginal sigmas
- Walkers count matches the ratio setting (walkers ratio 50 × 7 parameters = 350 walkers) and retained sample count matches the target (e.g. 87,500)
- Autocorrelation time from acor confirms the retained samples yield an effective independent sample size consistent with the assumed tolerance

## Limitations

- Scaling saturates once more processes are available than walkers, causing processes and their spawned threads to idle — oversubscribing cores does not speed up sampling or improve convergence checks.
- A doubling of cores incurs an ~11% scaling overhead (alpha = 0.89), so wall-time-limited sample budgets are not perfectly linear in resources.
- The WMAP data must be accessible on the filesystem and CFITSIO installed; the current distribution is only tested with Intel's ifort compiler and mkl libraries.
- The verification tolerance (~3.4% of sigma) is valid only for the specific sample sizes compared (e.g. 87,500 vs 35,000); different sample sizes require re-deriving the expected statistical error.

## Evidence

- [discussion] External data dependency for likelihood evaluation: [evidence span withheld — anchor status: not_found]
- [readme] Sampler architecture grounding the posterior comparison: [evidence span withheld — anchor status: not_found]
- [readme] Likelihood/core modules must be installed separately: [evidence span withheld — anchor status: not_found]
- [discussion] Chain construction with parameter bounds: [evidence span withheld — anchor status: not_found]
