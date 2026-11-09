---
name: multiwavelength-photometry-assembly
description: Use when building an ultraviolet-to-millimetre photometric data table
  for galaxies whose SEDs will be fitted with radiative transfer models, and when
  mid-IR spectra (e.g. Spitzer) must be included at matched spectral resolution.
when_to_use_negative:
- The analysis goal is SED fitting with flexible energy-balance templates rather than
  exclusive radiative transfer model libraries — SMART was benchmarked as comparable
  in runtime to energy balance methods but fits exclusively with radiative transfer
  models.
- No pre-computed radiative transfer library exists at a resolution matching the available
  spectroscopy, so spectra cannot be resampled for joint fitting.
- Only spectroscopic data are available without the ultraviolet-to-millimetre photometry
  that defines the fitting basis.
tools:
- name: SMART
  role: Bayesian MCMC SED-fitting code into which the assembled photometry and resampled
    spectra are loaded and fitted with four radiative transfer model libraries
  repo: https://github.com/ch-var/SMART.git
- name: Spitzer
  role: Source of mid-infrared spectroscopy (HERUS ULIRG spectra) resampled to model-library
    resolution and included in the fitting
- name: Python 3.10
  role: Implementation language/runtime (Anaconda package) in which SMART is developed
    and executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:6e57d181519fd834fb5de113d976eb95888774e963eb4819c9b346398472d0e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART.git
  tools:
  - Python 3.10
  - SMART
  - Spitzer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multiwavelength-photometry-assembly

## Summary

Assemble ultraviolet-to-millimetre multiwavelength photometry for galaxies (demonstrated on the 42 local ULIRGs of the HERUS sample) and optionally include mid-infrared spectroscopy, such as Spitzer spectra, resampled to the spectral resolution of pre-computed radiative transfer model libraries, so the combined data can be fitted with SMART's Bayesian MCMC SED-fitting code.

## When to use

Use when building an ultraviolet-to-millimetre photometric data table for galaxies whose SEDs will be fitted with radiative transfer models, and when mid-IR spectra (e.g. Spitzer) must be included at matched spectral resolution.

## When NOT to use

- The analysis goal is SED fitting with flexible energy-balance templates rather than exclusive radiative transfer model libraries — SMART was benchmarked as comparable in runtime to energy balance methods but fits exclusively with radiative transfer models.
- No pre-computed radiative transfer library exists at a resolution matching the available spectroscopy, so spectra cannot be resampled for joint fitting.
- Only spectroscopic data are available without the ultraviolet-to-millimetre photometry that defines the fitting basis.

## Inputs

- Ultraviolet-to-millimetre multiwavelength photometry for each target galaxy (demonstrated on 42 local ULIRGs from the HERUS sample)
- Mid-infrared spectroscopy (e.g. Spitzer spectroscopy data of the HERUS ULIRGs)
- Pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust) against which spectral resolution must be matched

## Outputs

- Photometry-plus-spectra input tables ready for SMART's Bayesian MCMC fitting
- Spitzer spectra resampled to the spectral resolution of the radiative transfer model libraries
- Per-galaxy best-fit SEDs and posterior distributions of fitted physical parameters

## How to apply

Load the multiwavelength ultraviolet-to-millimetre photometry for each galaxy into SMART (Python 3.10, Anaconda package). If spectroscopy is available (as with the Spitzer spectroscopy data of the HERUS ULIRGs), resample the spectra to the spectral resolution of the pre-computed radiative transfer model libraries so they can be jointly fitted with the photometry. Configure SMART's four radiative transfer libraries describing the starburst, AGN torus, host galaxy, and polar dust components, then run the Bayesian Markov chain Monte Carlo fit for each galaxy. The rationale for including the resampled spectra is that they constrain mid-IR emission (e.g. AGN torus and starburst contributions) that photometry alone cannot; SMART nevertheless runs in comparable time to popular energy balance methods despite fitting exclusively with radiative transfer models.

## Related tools

- **SMART** (Bayesian MCMC SED-fitting code into which the assembled photometry and resampled spectra are loaded and fitted with four radiative transfer model libraries) — https://github.com/ch-var/SMART.git
- **Spitzer** (Source of mid-infrared spectroscopy (HERUS ULIRG spectra) resampled to model-library resolution and included in the fitting)
- **Python 3.10** (Implementation language/runtime (Anaconda package) in which SMART is developed and executed)

## Examples

```
git clone https://github.com/ch-var/SMART.git
```

## Evaluation signals

- Best-fit SEDs and posterior parameter tables are produced for all 42 HERUS ULIRGs.
- Each best-fit SED reproduces both the observed ultraviolet-to-millimetre photometry and the included Spitzer spectroscopy.
- Spitzer spectra are resampled to the spectral resolution of the pre-computed radiative transfer model libraries before fitting.
- All four component libraries (starburst, AGN torus, host galaxy, polar dust) are configured and contribute to the fit.
- Runtime remains comparable to popular energy balance methods despite exclusive use of radiative transfer models.

## Limitations

- SMART fits SEDs exclusively with radiative transfer models from four pre-computed library types; it does not offer energy-balance-style template flexibility.
- Spectroscopy must be resampled to the spectral resolution of the pre-computed model libraries, which constrains which spectra can be included.
- The code is developed in Python 3.10 (Anaconda package) and distributed as OS-specific archives (Windows, Mac, Linux); no changelog was found in the repository discussion.
- SMART is licensed under GPL-3.0 (Copyright (c) 2024–2026 Charalambia Varnava).

## Evidence

- [abstract] Spitzer spectra included at matched resolution: "The Spitzer spectroscopy data of the HERUS ULIRGs are included in the fitting"
- [readme] UV-to-mm fitting with four RT libraries: "SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries"
- [intro] Implementation environment: [evidence span withheld — anchor status: not_found]
- [abstract] Runtime comparable to energy balance methods: "although it fits SEDs exclusively with radiative transfer models, it takes comparable time to popular energy balance methods to run"
- [abstract] HERUS test sample: "test it by fitting the multiwavelength SEDs of the 42 local ultraluminous infrared galaxies (ULIRGs) that constitute the HERschel Ultraluminous Infrared Galaxy Survey (HERUS) sample"
