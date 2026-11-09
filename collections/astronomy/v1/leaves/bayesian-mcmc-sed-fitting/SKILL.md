---
name: bayesian-mcmc-sed-fitting
description: Use when you need to fit multiwavelength (UV-to-millimetre) galaxy SEDs
  with physically motivated radiative transfer models via Bayesian MCMC, especially
  for ULIRGs or AGN-hosting galaxies, rather than empirical energy balance templates.
when_to_use_negative:
- You need a quick empirical/parametric fit and do not have the pre-computed radiative
  transfer libraries or the compute setup to run Bayesian MCMC
- The SED falls outside the ultraviolet-to-millimetre range covered by SMART's four
  model libraries (starburst, AGN torus, host galaxy, polar dust)
- You require a fit that does not exclusively use radiative transfer models — SMART
  fits SEDs 'exclusively with radiative transfer models' by design
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3393
tools:
- name: SMART
  role: Open-source SED fitting code implementing the Bayesian Markov chain Monte
    Carlo fit with radiative transfer model libraries
  repo: https://github.com/ch-var/SMART.git
- name: Python
  role: Language in which SMART is developed (Python 3.10, Anaconda package); used
    to load SED inputs, resample spectra, and extract result tables
  repo: https://github.com/ch-var/SMART.git
- name: Spitzer
  role: Source of spectroscopy data of the HERUS ULIRGs included in the fitting at
    matched spectral resolution
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:671e61a40563cfb46379c04fb5f5daa92cd5b2ed430554616a415e994262db01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART.git
  tools:
  - Python
  - SMART
  - Spitzer
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3393
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# bayesian-mcmc-sed-fitting

## Summary

Fit galaxy ultraviolet-to-millimetre spectral energy distributions with SMART, a Bayesian Markov chain Monte Carlo code that uses exclusively pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust), and can incorporate spectra such as Spitzer spectroscopy resampled to the model resolution.

## When to use

Use when you need to fit multiwavelength (UV-to-millimetre) galaxy SEDs with physically motivated radiative transfer models via Bayesian MCMC, especially for ULIRGs or AGN-hosting galaxies, rather than empirical energy balance templates.

## When NOT to use

- You need a quick empirical/parametric fit and do not have the pre-computed radiative transfer libraries or the compute setup to run Bayesian MCMC
- The SED falls outside the ultraviolet-to-millimetre range covered by SMART's four model libraries (starburst, AGN torus, host galaxy, polar dust)
- You require a fit that does not exclusively use radiative transfer models — SMART fits SEDs 'exclusively with radiative transfer models' by design

## Inputs

- Multiwavelength ultraviolet-to-millimetre photometry of galaxies (e.g. the 42 local ULIRGs of the HERUS sample)
- Spectroscopy data (e.g. Spitzer spectroscopy of the HERUS ULIRGs), resampled to the spectral resolution of the pre-computed radiative transfer model libraries
- Pre-computed radiative transfer model libraries for the starburst, AGN torus, host galaxy, and polar dust components

## Outputs

- Best-fit SED per galaxy, reproducing the observed photometry and any included spectroscopy
- Posterior distributions of the fitted physical parameters per galaxy, collected into result tables

## How to apply

Install SMART (developed in Python 3.10, Anaconda package; GPL-3.0) from https://github.com/ch-var/SMART.git for your operating system and consult the SMART User Manual. Load the multiwavelength ultraviolet-to-millimetre photometry of the target galaxies; if spectroscopy is available (e.g. Spitzer spectroscopy of HERUS ULIRGs), resample it to the spectral resolution of the pre-computed radiative transfer model libraries so it can be included in the fitting. Configure the four model libraries describing the starburst, AGN torus, host galaxy, and polar dust components, then execute the SMART Bayesian MCMC fit per galaxy. Extract, per galaxy, the best-fit SED and the posterior distributions of the fitted physical parameters into result tables; as a benchmark, SMART takes comparable time to popular energy balance methods despite fitting exclusively with radiative transfer models.

## Related tools

- **SMART** (Open-source SED fitting code implementing the Bayesian Markov chain Monte Carlo fit with radiative transfer model libraries) — https://github.com/ch-var/SMART.git
- **Python** (Language in which SMART is developed (Python 3.10, Anaconda package); used to load SED inputs, resample spectra, and extract result tables) — https://github.com/ch-var/SMART.git
- **Spitzer** (Source of spectroscopy data of the HERUS ULIRGs included in the fitting at matched spectral resolution)

## Examples

```
git clone https://github.com/ch-var/SMART.git; # then follow assets/SMART_User_Manual.pdf to run SMART v1.0 (SMART_v1.0.7z) on galaxy photometry + resampled spectra
```

## Evaluation signals

- Best-fit SEDs and posterior parameter tables are produced for all 42 HERUS ULIRGs in the sample
- Each best-fit SED reproduces both the observed photometry and the included Spitzer spectroscopy
- Spectroscopy was resampled to the spectral resolution of the radiative transfer model libraries before fitting
- Runtime per galaxy / full sample is of the same order as a popular energy balance method (SMART 'takes comparable time to popular energy balance methods to run')

## Limitations

- SMART currently fits exclusively with four types of pre-computed radiative transfer libraries (starburst, AGN torus, host galaxy, polar dust); SEDs outside what these libraries describe may not be captured
- The test on the HERUS sample used Spitzer spectroscopy at a resolution matched to the radiative transfer models; spectra at other resolutions must be resampled first
- No changelog found in the repository, so version-to-version behaviour changes are not documented

## Evidence

- [abstract] SMART fits UV-to-mm SEDs exclusively with radiative transfer libraries via Bayesian MCMC: "SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries, which describe the starburst,"
- [abstract] Spitzer spectroscopy included at matched spectral resolution: "The Spitzer spectroscopy data of the HERUS ULIRGs are included in the fitting"
- [abstract] Runtime comparable to energy balance methods: "although it fits SEDs exclusively with radiative transfer models, it takes comparable time to popular energy balance methods to run"
- [intro] SMART developed in Python 3.10: [evidence span withheld — anchor status: not_found]
- [readme] GitHub availability and license: "SMART is a publicly available open-source spectral energy distribution (SED) fitting code. Implementing a Bayesian Markov chain Monte Carlo method"
