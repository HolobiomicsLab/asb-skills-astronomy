---
name: infrared-spectral-rebinning
description: Use when mid/far-infrared spectroscopy (e.g. Spitzer IRS) must be included
  in an SED fit whose model library components (starburst, AGN torus, host galaxy,
  polar dust) are pre-computed on a coarser spectral resolution grid.
when_to_use_negative:
- The spectroscopy is already on (or coarser than) the radiative transfer model library's
  wavelength grid — convolution/rebinning is unnecessary
- You are fitting photometry-only SEDs without any spectral data; the rebinning step
  is specific to incorporating spectra into the SMART fit
- You need to fit high-resolution spectral features at their native resolution, since
  rebinning onto the model library grid deliberately degrades the spectral resolution
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3321
tools:
- name: SMART
  role: SED fitting code that consumes the resolution-matched Spitzer spectra alongside
    photometry, fitting UV-to-mm SEDs with Bayesian MCMC over four radiative transfer
    model libraries
  repo: https://github.com/ch-var/SMART.git
- name: Python
  role: Python 3.10 (Anaconda package) is the development environment in which the
    Spitzer spectra are loaded, inspected, masked, convolved, and rebinned
  repo: https://github.com/ch-var/SMART.git
- name: Spitzer
  role: Source of the mid-infrared spectroscopy of the HERUS ULIRGs that is preprocessed
    to the model library resolution before fitting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:a1f56408ba41916ce36b4d83dcbb02f0c72a5c4a7c88419ce95dc91c00ab89ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART.git
  tools:
  - Python
  - SMART
  - Spitzer
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3321
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Infrared spectral rebinning

## Summary

Convolve and rebin mid-infrared spectroscopy (e.g. Spitzer IRS spectra of the 42 HERUS ULIRGs) onto the coarser wavelength grid of pre-computed radiative transfer model libraries so the spectra can be jointly fitted with photometric SED points in SMART, propagating flux uncertainties through the rebinning.

## When to use

Use when mid/far-infrared spectroscopy (e.g. Spitzer IRS) must be included in an SED fit whose model library components (starburst, AGN torus, host galaxy, polar dust) are pre-computed on a coarser spectral resolution grid.

## When NOT to use

- The spectroscopy is already on (or coarser than) the radiative transfer model library's wavelength grid — convolution/rebinning is unnecessary
- You are fitting photometry-only SEDs without any spectral data; the rebinning step is specific to incorporating spectra into the SMART fit
- You need to fit high-resolution spectral features at their native resolution, since rebinning onto the model library grid deliberately degrades the spectral resolution

## Inputs

- Spitzer spectroscopy files for each of the 42 HERUS ULIRGs (wavelength, flux, and uncertainty arrays)
- Pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust) defining the target resolution grid

## Outputs

- Resolution-matched spectra assembled into a single multi-extension FITS file with one extension per galaxy (42 extensions for the HERUS sample)
- Flagged list of galaxies whose spectra fail to load or whose wavelength grids do not align with the model library

## How to apply

Load each Spitzer spectroscopy file (wavelength, flux, uncertainty arrays) for the 42 HERUS ULIRGs in Python 3.10 within the SMART environment, first inspecting and masking bad or negative-flux channels that would bias the fit. Convolve and rebin each spectrum onto the coarser spectral resolution grid of the pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust components) used by SMART, propagating the flux uncertainties through the rebinning. Assemble the resolution-matched spectra for all sources into a single multi-extension FITS file with one extension per source. Finally run a sanity check through SMART's input reader to confirm the preprocessed spectra parse correctly and their wavelength grids align with the model library coverage; the rationale is that fitting raw-resolution spectra against a coarser model grid would mismatch the data and model sampling.

## Related tools

- **SMART** (SED fitting code that consumes the resolution-matched Spitzer spectra alongside photometry, fitting UV-to-mm SEDs with Bayesian MCMC over four radiative transfer model libraries) — https://github.com/ch-var/SMART.git
- **Python** (Python 3.10 (Anaconda package) is the development environment in which the Spitzer spectra are loaded, inspected, masked, convolved, and rebinned) — https://github.com/ch-var/SMART.git
- **Spitzer** (Source of the mid-infrared spectroscopy of the HERUS ULIRGs that is preprocessed to the model library resolution before fitting)

## Evaluation signals

- The output multi-extension FITS file exists and contains exactly 42 spectra extensions, one per HERUS ULIRG
- Every spectrum's wavelength grid lies within the radiative transfer model library's resolution/coverage
- SMART's input reader parses every preprocessed spectrum without errors and the grids align with the model library wavelength coverage
- Galaxies whose spectra fail to load are flagged rather than silently omitted from the assembled FITS file
- Bad/negative-flux channels are masked before rebinning so they do not bias the subsequent SED fit

## Limitations

- Rebinning deliberately matches (degrades) the Spitzer spectral resolution to the pre-computed model library resolution, so fine spectral features cannot be exploited in the fit
- Uncertainty propagation through convolution/rebinning must be handled explicitly; the workflow prescribes propagating flux uncertainties through the rebinning
- The skill was demonstrated on the 42 local ULIRGs of the HERUS sample; applying it to other samples requires adapting to that data's channel defects and masking

## Evidence

- [abstract] Spitzer spectroscopy included at model-matched resolution: "The Spitzer spectroscopy data of the HERUS ULIRGs are included in the fitting"
- [abstract] Four pre-computed radiative transfer libraries define the target grid: "radiative transfer models that currently constitute four types of pre-computed libraries, which describe the starburst,"
- [intro] SMART developed in Python 3.10: [evidence span withheld — anchor status: not_found]
- [abstract] HERUS sample of 42 local ULIRGs: "test it by fitting the multiwavelength SEDs of the 42 local ultraluminous infrared galaxies (ULIRGs) that constitute the HERschel Ultraluminous Infrared Galaxy Survey (HERUS) sample"
- [readme] SMART is open-source SED fitting code with Bayesian MCMC: [evidence span withheld — anchor status: not_found]
