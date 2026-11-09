---
name: radiative-transfer-model-library-configuration
description: Use when fitting ultraviolet-to-millimetre galaxy SEDs with SMART and
  you must configure its four pre-computed radiative transfer libraries (starburst,
  AGN torus, host galaxy, polar dust), including matching Spitzer spectroscopy resolution
  to the models.
when_to_use_negative:
- The input spectra cannot be resampled to the spectral resolution of the pre-computed
  radiative transfer model libraries, so spectroscopy cannot be legitimately included
  in the fit
- The galaxy SED requires components beyond the four SMART library types (starburst,
  AGN torus, host galaxy, polar dust) or you need a non-radiative-transfer (e.g. energy-balance)
  fitting approach
tools:
- name: SMART
  role: Open-source SED fitting code whose four radiative transfer libraries are configured
    and run with a Bayesian Markov chain Monte Carlo method
  repo: https://github.com/ch-var/SMART.git
- name: Python
  role: SMART is developed in Python 3.10 (Anaconda package); the configuration and
    fitting are executed in this environment
- name: Spitzer
  role: Source of the spectroscopy data included in the fitting at a resolution matched
    to the radiative transfer models
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:df9c593e9a88d5b72d2ea3051d79920e70b7180da3f924fc97eb89baf4461b41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART.git
  tools:
  - Python
  - SMART
  - Spitzer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# radiative-transfer-model-library-configuration

## Summary

Configures SMART's four pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust) that exclusively define galaxy ultraviolet-to-millimetre SEDs prior to the Bayesian Markov chain Monte Carlo fit.

## When to use

Use when fitting ultraviolet-to-millimetre galaxy SEDs with SMART and you must configure its four pre-computed radiative transfer libraries (starburst, AGN torus, host galaxy, polar dust), including matching Spitzer spectroscopy resolution to the models.

## When NOT to use

- The input spectra cannot be resampled to the spectral resolution of the pre-computed radiative transfer model libraries, so spectroscopy cannot be legitimately included in the fit
- The galaxy SED requires components beyond the four SMART library types (starburst, AGN torus, host galaxy, polar dust) or you need a non-radiative-transfer (e.g. energy-balance) fitting approach

## Inputs

- Multiwavelength ultraviolet-to-millimetre photometry of galaxies (e.g. the 42 HERUS ULIRGs)
- Spitzer spectroscopy data resampled to the spectral resolution of the radiative transfer model libraries
- SMART's four pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust)

## Outputs

- Best-fit SED per galaxy reproducing the observed photometry and included Spitzer spectroscopy
- Posterior distributions of the fitted physical parameters per galaxy, collected into result tables

## How to apply

Load the multiwavelength ultraviolet-to-millimetre photometry (and, if available, spectroscopy such as the Spitzer data of the HERUS ULIRGs) into SMART (Python 3.10, Anaconda package). Resample any included spectroscopy to the spectral resolution of the pre-computed radiative transfer model libraries so the spectra can be used in the fit — SMART fits SEDs exclusively with radiative transfer models, not energy-balance approximations. Configure the four library types that describe the starburst, AGN torus, host galaxy, and polar dust components; all four are used together to build the model SED. Execute the Bayesian Markov chain Monte Carlo fit, then extract the best-fit SED and posterior distributions of the fitted physical parameters. SMART runs in comparable time to popular energy balance methods despite using radiative transfer models exclusively.

## Related tools

- **SMART** (Open-source SED fitting code whose four radiative transfer libraries are configured and run with a Bayesian Markov chain Monte Carlo method) — https://github.com/ch-var/SMART.git
- **Python** (SMART is developed in Python 3.10 (Anaconda package); the configuration and fitting are executed in this environment)
- **Spitzer** (Source of the spectroscopy data included in the fitting at a resolution matched to the radiative transfer models)

## Evaluation signals

- All four library types (starburst, AGN torus, host galaxy, polar dust) are configured and present in the model SED before the MCMC run
- Included Spitzer spectroscopy has been resampled to the spectral resolution of the radiative transfer model libraries prior to fitting
- Best-fit SEDs and posterior parameter tables are produced for every galaxy in the sample (e.g. all 42 HERUS ULIRGs)
- Each best-fit SED reproduces both the observed ultraviolet-to-millimetre photometry and the included Spitzer spectroscopy
- Runtime remains comparable to popular energy balance methods despite fitting exclusively with radiative transfer models

## Limitations

- SMART fits SEDs exclusively with radiative transfer models and its pre-computed libraries currently cover only four component types (starburst, AGN torus, host galaxy, polar dust)
- Spectroscopy (e.g. Spitzer data) must be resampled to the spectral resolution of the model libraries before inclusion in the fitting
- No changelog found for the repository, so version history and changes are not documented

## Evidence

- [abstract] SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries: "SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries"
- [abstract] The Spitzer spectroscopy data of the HERUS ULIRGs are included in the fitting: "The Spitzer spectroscopy data of the HERUS ULIRGs are included in the fitting"
- [intro] The code is developed in PYTHON 3.10 (Anaconda package).: [evidence span withheld — anchor status: not_found]
- [other] Resample the Spitzer spectroscopy data of the HERUS ULIRGs to the spectral resolution of the pre-computed radiative transfer model libraries so the spectra can be included in the fitting.: [evidence span withheld — anchor status: not_found]
