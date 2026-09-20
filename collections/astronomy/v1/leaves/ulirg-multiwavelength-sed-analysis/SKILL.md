---
name: ulirg-multiwavelength-sed-analysis
description: Use when fitting multiwavelength galaxy SEDs (UV to millimetre, including
  Spitzer mid-IR spectroscopy) for ULIRGs or AGN-hosted systems where physically motivated
  radiative transfer decomposition is preferred over empirical energy balance.
when_to_use_negative:
- The analysis requires empirical energy balance SED fitting or semi-analytic models
  rather than radiative transfer libraries — SMART fits exclusively with pre-computed
  RT models.
- The targets are not galaxies with UV-to-millimetre SED coverage; the code was developed
  and tested on ULIRG SEDs (HERUS sample) with Spitzer spectroscopy.
- No pre-computed library among the four types (starburst, AGN torus, host galaxy,
  polar dust) matches the physical components needed for the source.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SMART
  role: 'Primary SED-fitting code: Bayesian MCMC fitting of UV-to-mm SEDs exclusively
    with radiative transfer model libraries'
  repo: https://github.com/ch-var/SMART.git
- name: Python
  role: Implementation language of SMART (Python 3.10, Anaconda) and for assembling
    runtime comparison tables
  repo: https://github.com/ch-var/SMART.git
- name: Spitzer
  role: Source of mid-infrared spectroscopy of the HERUS ULIRGs included in the fitting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:2fda2cb00d42315e0371f032f674a4b961978d4b3d962f954c7f1ba27b37f961
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART.git
  tools:
  - Python
  - SMART
  - Spitzer
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ulirg-multiwavelength-sed-analysis

## Summary

Fit ultraviolet-to-millimetre spectral energy distributions of ULIRGs using SMART, a Bayesian MCMC code that relies exclusively on pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust) rather than energy balance; demonstrated on the 42-galaxy HERUS sample with Spitzer spectroscopy.

## When to use

Use when fitting multiwavelength galaxy SEDs (UV to millimetre, including Spitzer mid-IR spectroscopy) for ULIRGs or AGN-hosted systems where physically motivated radiative transfer decomposition is preferred over empirical energy balance.

## When NOT to use

- The analysis requires empirical energy balance SED fitting or semi-analytic models rather than radiative transfer libraries — SMART fits exclusively with pre-computed RT models.
- The targets are not galaxies with UV-to-millimetre SED coverage; the code was developed and tested on ULIRG SEDs (HERUS sample) with Spitzer spectroscopy.
- No pre-computed library among the four types (starburst, AGN torus, host galaxy, polar dust) matches the physical components needed for the source.

## Inputs

- Multiwavelength photometric SED points for each galaxy (ultraviolet to millimetre)
- Spitzer mid-infrared spectroscopy of the HERUS ULIRGs (resolution matched to the RT model libraries)
- SMART pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust)
- SMART v1.0 package (Python 3.10, Anaconda) downloaded per operating system from GitHub

## Outputs

- Bayesian MCMC-fitted SED decomposition per galaxy into starburst, AGN torus, host galaxy, and polar dust components
- Per-object and full-sample wall-clock runtime measurements (e.g. runtime_comparison.csv for the 42 HERUS galaxies)
- Fitted SEDs usable for galaxy evolution studies, including in the JWST era

## How to apply

Load the multiwavelength SED photometry plus Spitzer spectroscopy for each galaxy (e.g. the 42 HERUS ULIRGs) into SMART, which is a Python 3.10 (Anaconda) open-source code distributed as OS-specific archives via GitHub. Run SMART's Bayesian Markov chain Monte Carlo fitting, which decomposes each SED exclusively with four types of pre-computed radiative transfer libraries describing the starburst, AGN torus, host galaxy, and polar dust components; match the Spitzer spectroscopy resolution to the radiative transfer models as done in the paper. Consult the SMART_User_Manual.pdf for run instructions and choose the library combination appropriate to the source's starburst/AGN composition. Benchmark wall-clock time if comparing against energy balance codes — the paper reports SMART runs in comparable time to popular energy balance methods despite using RT models. Assemble per-object fit outputs (e.g. runtime and fitted SED tables) programmatically with Python for downstream comparison.

## Related tools

- **SMART** (Primary SED-fitting code: Bayesian MCMC fitting of UV-to-mm SEDs exclusively with radiative transfer model libraries) — https://github.com/ch-var/SMART.git
- **Python** (Implementation language of SMART (Python 3.10, Anaconda) and for assembling runtime comparison tables) — https://github.com/ch-var/SMART.git
- **Spitzer** (Source of mid-infrared spectroscopy of the HERUS ULIRGs included in the fitting)

## Examples

```
python smart_fit.py --sed herus_ulirg_sed.dat --spitzer_spec spitzer_ir.txt --libraries starburst,agn_torus,host,polar_dust
```

## Evaluation signals

- All 42 HERUS ULIRG SEDs converge through the MCMC fit with contributions from the four RT library types (starburst, AGN torus, host galaxy, polar dust).
- Spitzer spectroscopy is included in each fit at resolution matched to the radiative transfer models.
- Measured wall-clock time per galaxy and for the full sample is of the same order as reported runtimes of popular energy balance methods.
- A runtime comparison table (e.g. runtime_comparison.csv) contains 42 galaxy entries plus summary rows.
- Fitted SEDs span the ultraviolet-to-millimetre range claimed by the code.

## Limitations

- SMART depends on pre-computed radiative transfer libraries; only four library types (starburst, AGN torus, host galaxy, polar dust) currently exist, so novel physical components require new libraries.
- The comparison against energy balance methods is for ULIRG SED fitting; runtime parity may not transfer to other samples or configurations.
- No changelog is available for the repository, and versioned downloads are 7z archives per OS (SMART_v1.0.7z / SMART_v1.0_mac.7z).

## Evidence

- [abstract] SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries: "SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries"
- [abstract] it takes comparable time to popular energy balance methods to run: "although it fits SEDs exclusively with radiative transfer models, it takes comparable time to popular energy balance methods to run"
- [intro] The code is developed in PYTHON 3.10 (Anaconda package).: [evidence span withheld — anchor status: not_found]
- [abstract] The Spitzer spectroscopy data of the HERUS ULIRGs are included in the fitting: "The Spitzer spectroscopy data of the HERUS ULIRGs are included in the fitting"
- [readme] SMART is developed in Python and is available at https://github.com/ch-var/SMART.git: "SMART is developed in Python and is available at https://github.com/ch-var/SMART.git"
