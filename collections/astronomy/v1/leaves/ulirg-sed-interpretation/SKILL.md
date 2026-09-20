---
name: ulirg-sed-interpretation
description: Use when interpreting multiwavelength (UV-to-millimetre) photometry —
  and optionally mid-infrared Spitzer spectroscopy — of ULIRGs or dust-rich galaxies
  where physically motivated radiative transfer decomposition into starburst, AGN
  torus, host, and polar dust components is required.
when_to_use_negative:
- The target lacks ultraviolet-to-millimetre photometric coverage, so the four radiative
  transfer components cannot be constrained
- Only narrow-band photometry without mid-IR spectroscopy or far-IR constraints is
  available and the goal is a quick empirical SED classification rather than physical
  radiative transfer decomposition
- The analysis targets resolved spatial maps or line diagnostics rather than integrated
  galaxy SED fitting
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SMART
  role: Open-source SED fitting code performing the Bayesian MCMC fit exclusively
    with radiative transfer model libraries
  repo: https://github.com/ch-var/SMART.git
- name: Python 3.10
  role: Language/runtime in which SMART is developed and executed (Anaconda package)
- name: Spitzer
  role: Source of mid-infrared spectroscopy data of the HERUS ULIRGs included in the
    fitting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:b78fe393e07d413520b743e795d47255279793f7df61209af0cffaeb7097f187
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART.git
  tools:
  - Python 3.10
  - SMART
  - Spitzer
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ulirg-sed-interpretation

## Summary

Fits ultraviolet-to-millimetre SEDs of local ULIRGs (e.g. the 42-galaxy HERUS sample) with the open-source SMART code, using four pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust) and a Bayesian Markov chain Monte Carlo method, optionally including Spitzer spectroscopy resampled to the model spectral resolution.

## When to use

Use when interpreting multiwavelength (UV-to-millimetre) photometry — and optionally mid-infrared Spitzer spectroscopy — of ULIRGs or dust-rich galaxies where physically motivated radiative transfer decomposition into starburst, AGN torus, host, and polar dust components is required.

## When NOT to use

- The target lacks ultraviolet-to-millimetre photometric coverage, so the four radiative transfer components cannot be constrained
- Only narrow-band photometry without mid-IR spectroscopy or far-IR constraints is available and the goal is a quick empirical SED classification rather than physical radiative transfer decomposition
- The analysis targets resolved spatial maps or line diagnostics rather than integrated galaxy SED fitting

## Inputs

- Multiwavelength ultraviolet-to-millimetre photometry for each galaxy (e.g. the 42 HERUS ULIRGs)
- Spitzer spectroscopy data of the target ULIRGs (resampled to the radiative transfer model spectral resolution)
- SMART's four pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust)

## Outputs

- Best-fit SED per galaxy reproducing the observed photometry and included Spitzer spectroscopy
- Posterior distributions of the fitted physical parameters per galaxy, compiled into result tables

## How to apply

Load the galaxy's multiwavelength ultraviolet-to-millimetre photometry into SMART (Python 3.10, Anaconda) and, if mid-IR spectra are available, resample the Spitzer spectroscopy to the spectral resolution of the pre-computed radiative transfer model libraries so spectra can enter the fit alongside photometry. Configure SMART's four radiative transfer libraries describing the starburst, AGN torus, host galaxy, and polar dust components, then run the Bayesian Markov chain Monte Carlo fit for each object. Unlike energy-balance codes, SMART fits exclusively with radiative transfer models yet runs in comparable time to popular energy balance methods. Extract the best-fit SED and posterior distributions of the fitted physical parameters into result tables, and validate that the best-fit SED reproduces both the observed photometry and the included Spitzer spectroscopy.

## Related tools

- **SMART** (Open-source SED fitting code performing the Bayesian MCMC fit exclusively with radiative transfer model libraries) — https://github.com/ch-var/SMART.git
- **Python 3.10** (Language/runtime in which SMART is developed and executed (Anaconda package))
- **Spitzer** (Source of mid-infrared spectroscopy data of the HERUS ULIRGs included in the fitting)

## Evaluation signals

- Best-fit SEDs and posterior parameter tables are produced for all 42 HERUS ULIRGs
- Each best-fit SED reproduces the observed photometry of the galaxy
- Each best-fit SED also reproduces the included (resolution-matched) Spitzer spectroscopy
- Run time remains comparable to popular energy balance methods despite fitting exclusively with radiative transfer models

## Limitations

- Radiative transfer model libraries are pre-computed and currently cover only four component types (starburst, AGN torus, host galaxy, polar dust), so components outside these libraries cannot be represented
- Spitzer spectroscopy must be resampled to the spectral resolution of the pre-computed radiative transfer models before inclusion, imposing a resolution ceiling on spectral information
- No changelog was found for the repository, limiting traceability of code version changes

## Evidence

- [abstract] Exclusive radiative-transfer fitting with four libraries and Bayesian MCMC: "SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries, which describe the starburst,"
- [abstract] Spitzer spectroscopy included at matched spectral resolution: "The Spitzer spectroscopy data of the HERUS ULIRGs are included in the fitting"
- [intro] Development environment and Bayesian MCMC method: [evidence span withheld — anchor status: not_found]
- [abstract] Runtime comparable to energy balance methods: "although it fits SEDs exclusively with radiative transfer models, it takes comparable time to popular energy balance methods to run"
- [readme] Software availability and license: [evidence span withheld — anchor status: not_found]
