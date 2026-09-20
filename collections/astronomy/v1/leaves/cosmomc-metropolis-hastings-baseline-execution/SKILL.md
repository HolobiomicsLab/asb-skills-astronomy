---
name: cosmomc-metropolis-hastings-baseline-execution
description: 'Reference for establishing a Metropolis–Hastings baseline: when comparing
  a new MCMC sampler''s efficiency (autocorrelation time per parameter) against CosmoMC
  on identical cosmological parameter spaces and likelihoods.'
when_to_use_negative:
- When the goal is parallel wall-time scaling of an ensemble sampler across many cores/nodes
  — CosmoMC Metropolis–Hastings is dominated by overheads and is not the tool for
  the scaling experiment itself, only the baseline.
- When only a single sampler's chains are available with no matched parameter space
  or likelihood — autocorrelation-time comparisons require the same WMAP 7 parameter
  space and likelihood in both runs.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CosmoMC
  role: Fortran Metropolis–Hastings sampler run in default and fine-tuned configurations
    on the WMAP 7 parameter space to provide the baseline chains
- name: CAMB
  role: Boltzmann integrator underlying the CosmoMC likelihood computation
- name: acor
  role: Python wrapper (pip install acor) of Jonathan Goodman's ACOR routine used
    to estimate the exponential autocorrelation time of each parameter chain
  repo: https://github.com/dfm/acor
- name: CosmoHammer
  role: Goodman–Weare emcee-based sampler whose tau values are compared against the
    CosmoMC baseline
  repo: http://github.com/cosmo-ethz/CosmoHammer
- name: CosmoHammerPlugins
  role: Provides the WMAP likelihood and CAMB core modules defining the shared parameter
    space
  repo: http://github.com/cosmo-ethz/CosmoHammerPlugins
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.06.003
    title: 'CosmoHammer: Cosmological parameter estimation with the MCMC Hammer'
  source_content_hash: sha256:7b87ee16e4df0a819f42cf5cc5309e1a91c691e2aa3a7cc88e0bde63724adb09
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cosmomc-metropolis-hastings-baseline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run the Fortran CosmoMC Metropolis–Hastings sampler (default and fine-tuned configurations) on the same WMAP 7 parameter space used by CosmoHammer, then estimate per-parameter autocorrelation times to serve as the performance baseline for the Goodman–Weare emcee ensemble sampler.

## When to use

Reference for establishing a Metropolis–Hastings baseline: when comparing a new MCMC sampler's efficiency (autocorrelation time per parameter) against CosmoMC on identical cosmological parameter spaces and likelihoods.

## When NOT to use

- When the goal is parallel wall-time scaling of an ensemble sampler across many cores/nodes — CosmoMC Metropolis–Hastings is dominated by overheads and is not the tool for the scaling experiment itself, only the baseline.
- When only a single sampler's chains are available with no matched parameter space or likelihood — autocorrelation-time comparisons require the same WMAP 7 parameter space and likelihood in both runs.

## Inputs

- WMAP 7-year likelihood data and WMAP likelihood code (Fortran, requires CFITSIO and WMAP data on the local filesystem)
- CAMB Boltzmann integrator as the theory code behind the likelihood
- CosmoMC configuration: sampled parameter arrays (start centre, min, max, start width), e.g. H0 [70, 40, 100, 3], Omega_b, Omega_cdm, tau, n_s, logA, sigma8
- MCMC chain output time series per parameter (from default and fine-tuned CosmoMC runs)

## Outputs

- Per-parameter exponential autocorrelation times tau for the fine-tuned (and default) CosmoMC Metropolis–Hastings chains
- Side-by-side comparison table of tau for CosmoMC vs. CosmoHammer/emcee (Table 2 style), e.g. fine-tuned CosmoMC tau ≈ 13.8–18.4 vs. CosmoHammer tau ≈ 43.4–48.2
- CSV of autocorrelation times suitable for validation checks

## How to apply

Configure CosmoMC, which chains the Metropolis–Hastings algorithm to the CAMB Boltzmann integrator, to sample the same WMAP 7-year parameter space as CosmoHammer (seven parameters including H0, Omega_b, Omega_cdm, tau, n_s, logA, sigma8, defined via start centre/min/max/start-width arrays). Run two configurations: the default settings and a fine-tuned variant, so the comparison isolates the effect of tuning proposal steps. For each resulting chain, compute the exponential autocorrelation time per parameter using the Goodman–Weare Appendix C estimator implemented in the acor package (github.com/dfm/acor), called as tau, mean, sigma = acor.acor(x) on each parameter time series. Tabulate tau side by side for CosmoMC (both configurations) and CosmoHammer; in the paper the fine-tuned CosmoMC achieves tau ≈ 13.8–18.4 versus CosmoHammer's τ ≈ 43.4–48.2, so the baseline validates that CosmoHammer's ensemble sampler is competitive despite higher per-parameter autocorrelation (it compensates via cheap, parallelisable chain steps).

## Related tools

- **CosmoMC** (Fortran Metropolis–Hastings sampler run in default and fine-tuned configurations on the WMAP 7 parameter space to provide the baseline chains)
- **CAMB** (Boltzmann integrator underlying the CosmoMC likelihood computation)
- **acor** (Python wrapper (pip install acor) of Jonathan Goodman's ACOR routine used to estimate the exponential autocorrelation time of each parameter chain) — https://github.com/dfm/acor
- **CosmoHammer** (Goodman–Weare emcee-based sampler whose tau values are compared against the CosmoMC baseline) — http://github.com/cosmo-ethz/CosmoHammer
- **CosmoHammerPlugins** (Provides the WMAP likelihood and CAMB core modules defining the shared parameter space) — http://github.com/cosmo-ethz/CosmoHammerPlugins

## Examples

```
import acor; tau, mean, sigma = acor.acor(chain[:, i])  # per-parameter autocorrelation time from a CosmoMC chain
```

## Evaluation signals

- Reproduced fine-tuned CosmoMC tau values fall in the ≈ 13.8–18.4 range across the seven sampled parameters, and self-tuning CosmoMC in ≈ 13.3–19.1, matching Table 2.
- CosmoHammer tau values fall in the ≈ 43.4–48.2 range, confirming the baseline ratio (CosmoHammer roughly 3× higher autocorrelation than fine-tuned CosmoMC).
- CSV output exists, parses, and contains tau values within the expected ranges for all seven parameters (H0, Omega_b, Omega_cdm, tau, n_s, logA, sigma8).
- acor returns three values (tau, mean, sigma) for every parameter time series without convergence warnings from the estimator.

## Limitations

- Metropolis–Hastings chains are serial: each step depends on the previous one, so the approach is dominated by overheads in parallelised settings and cannot exploit large-scale environments the way emcee can.
- CosmoMC/CAMB are Fortran90; the distribution has only been tested with Intel's ifort compiler and MKL libraries, and WMAP data plus CFITSIO must be present on the filesystem.
- Autocorrelation-time estimates depend on chain length and the exponential-tau estimator; short chains give noisy tau and can obscure the baseline comparison.

## Evidence

- [discussion] the WMAP data needs to be accessible on the filesystem and CFITSIO has to be installed.: [evidence span withheld — anchor status: not_found]
