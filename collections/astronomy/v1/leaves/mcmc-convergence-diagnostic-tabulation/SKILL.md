---
name: mcmc-convergence-diagnostic-tabulation
description: Use when you have posterior chains from ensemble (Goodman-Weare/emcee)
  or Metropolis-Hastings MCMC samplers and need per-parameter autocorrelation times
  to compare sampler convergence and efficiency side by side.
when_to_use_negative:
- Input is not MCMC chain time series data (e.g. you need raw likelihood values or
  best-fit points from particle swarm optimization rather than convergence diagnostics).
- Chains are too short or not yet mixed relative to the expected tau — the acor estimator
  requires sufficiently long, stationary series to yield reliable tau.
- The comparison target is wall-time scaling on distributed cores rather than per-parameter
  sampling efficiency.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: acor
  role: Python port of Jonathan Goodman's ACOR C++ routine; computes the exponential
    autocorrelation time tau, mean, and sigma for each per-parameter chain
  repo: https://github.com/dfm/acor
- name: CosmoHammer
  role: Generates the emcee (Goodman-Weare affine-invariant) posterior chains whose
    tau values are tabulated
  repo: https://github.com/cosmo-ethz/CosmoHammer
- name: CosmoHammerPlugins
  role: Supplies the CAMB core module and WMAP 7-year likelihood modules used to produce
    the chains
  repo: https://github.com/cosmo-ethz/CosmoHammerPlugins
- name: emcee
  role: Affine-invariant ensemble sampler embedded in CosmoHammer; its Goodman-Weare
    algorithm motivates the Appendix C tau estimator
  repo: https://github.com/cosmo-ethz/CosmoHammer
- name: CosmoMC
  role: Fortran Metropolis-Hastings baseline sampler whose fine-tuned and self-tuning
    runs provide comparison tau values
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:c6f30a830b8ec0f95ce1318fca860b95b3296b3dc041ae2497d055c3d67afd19
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cosmo-ethz/CosmoHammer
  repo_url: https://github.com/dfm/acor
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mcmc-convergence-diagnostic-tabulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates and tabulates per-parameter autocorrelation times (tau) from MCMC chains using the Goodman-Weare exponential autocorrelation estimator (the acor package), enabling quantitative comparison of sampler efficiency, e.g. CosmoHammer/emcee versus fine-tuned and self-tuning CosmoMC Metropolis-Hastings runs on the WMAP 7 likelihood.

## When to use

Use when you have posterior chains from ensemble (Goodman-Weare/emcee) or Metropolis-Hastings MCMC samplers and need per-parameter autocorrelation times to compare sampler convergence and efficiency side by side.

## When NOT to use

- Input is not MCMC chain time series data (e.g. you need raw likelihood values or best-fit points from particle swarm optimization rather than convergence diagnostics).
- Chains are too short or not yet mixed relative to the expected tau — the acor estimator requires sufficiently long, stationary series to yield reliable tau.
- The comparison target is wall-time scaling on distributed cores rather than per-parameter sampling efficiency.

## Inputs

- Posterior MCMC chains (per-parameter time series) from CosmoHammer/emcee runs on the WMAP 7 likelihood
- Posterior chains from CosmoMC Metropolis-Hastings runs (default and fine-tuned configurations) on the same WMAP 7 parameter space
- Parameter configuration array (start centre, min, max, start width) for the sampled cosmological parameters

## Outputs

- Per-parameter exponential autocorrelation times tau (via acor: tau, mean, sigma triples)
- Side-by-side comparison table of tau values for CosmoHammer and both CosmoMC configurations (Table 2 equivalent)

## How to apply

For each chain (CosmoHammer with emcee at walkers ratio 50, i.e. 350 walkers for the 7 sampled WMAP 7 parameters; plus default and fine-tuned CosmoMC runs), compute the exponential autocorrelation time per sampled cosmological parameter (H0, Omega_b, Omega_cdm, tau, n_s, logA, sigma8) with the Appendix C Goodman-Weare estimator as implemented in the Python acor package, which returns tau, mean, and sigma for each time series. Tabulate tau per parameter for all sampler configurations side by side (in this paper's Table 2) and validate against expected ranges: CosmoHammer tau ≈ 43.4–48.2, fine-tuned CosmoMC tau ≈ 13.8–18.4, self-tuning CosmoMC tau ≈ 13.3–19.1. The rationale is that tau measures how many steps are needed to generate an independent sample, so it directly quantifies the relative sampling efficiency of the affine-invariant ensemble sampler versus Metropolis-Hastings on the same WMAP 7 parameter space.

## Related tools

- **acor** (Python port of Jonathan Goodman's ACOR C++ routine; computes the exponential autocorrelation time tau, mean, and sigma for each per-parameter chain) — https://github.com/dfm/acor
- **CosmoHammer** (Generates the emcee (Goodman-Weare affine-invariant) posterior chains whose tau values are tabulated) — https://github.com/cosmo-ethz/CosmoHammer
- **CosmoHammerPlugins** (Supplies the CAMB core module and WMAP 7-year likelihood modules used to produce the chains) — https://github.com/cosmo-ethz/CosmoHammerPlugins
- **emcee** (Affine-invariant ensemble sampler embedded in CosmoHammer; its Goodman-Weare algorithm motivates the Appendix C tau estimator) — https://github.com/cosmo-ethz/CosmoHammer
- **CosmoMC** (Fortran Metropolis-Hastings baseline sampler whose fine-tuned and self-tuning runs provide comparison tau values)

## Examples

```
import acor; tau, mean, sigma = acor.acor(x)
```

## Evaluation signals

- Reproduced tau values for CosmoHammer fall in ≈ 43.4–48.2 per parameter, matching Table 2.
- Reproduced tau values fall in ≈ 13.8–18.4 for fine-tuned CosmoMC and ≈ 13.3–19.1 for self-tuning CosmoMC.
- The output CSV/table exists, parses, and contains one tau per sampled parameter (e.g. H0, Omega_b, Omega_cdm, tau, n_s, logA, sigma8).
- acor returns a (tau, mean, sigma) triple per time series, and walker count equals walkers_ratio × n_params (e.g. 50 × 7 = 350 for WMAP 7).

## Limitations

- Scaling saturates once more processes are available than walkers, causing processes and their spawned threads to idle (plateau observed at 64 nodes) — chains must be long enough that this does not bias tau.
- The current CosmoHammer distribution has only been tested with Intel's ifort compiler and mkl libraries; other Fortran compilers are untested.
- WMAP data must be accessible on the filesystem and CFITSIO installed; acor's only Python dependency is NumPy (including python-dev/python-numpy-dev on some systems).

## Evidence

- [discussion] The emcee sampler by Foreman-Mackey et al. (2012) turned out to be a good choice regarding those preconditions: [evidence span withheld — anchor status: not_found]
- [readme] This is a direct port of a C++ routine by Jonathan Goodman (NYU) called ACOR that estimates the autocorrelation time of time series data very quickly.: [evidence span withheld — anchor status: external_doc]
- [readme] tau, mean, sigma = acor.acor(x): [evidence span withheld — anchor status: external_doc]
