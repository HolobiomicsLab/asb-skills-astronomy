---
name: spitzer-spectra-resolution-matching
description: Use when adding low/medium-resolution space-based spectra (e.g. Spitzer
  IRS spectroscopy) to a radiative-transfer SED fit whose model libraries are sampled
  on a coarser wavelength grid, so spectra and models must be resolution-matched before
  fitting.
when_to_use_negative:
- When the data are purely photometric (UV-to-mm fluxes) with no spectroscopy to resample
  — SMART can be fit on photometry alone without this preprocessing
- When the input spectra are already on the radiative transfer model library's wavelength
  grid and resolution — rebinning would needlessly degrade them
- When using an energy balance SED-fitting code rather than SMART's radiative transfer
  model libraries, since the target grid is library-specific
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: SMART
  role: SED-fitting code whose pre-computed radiative transfer model libraries define
    the target spectral resolution; its input reader validates the preprocessed spectra
  repo: https://github.com/ch-var/SMART.git
- name: Python 3.10
  role: Language (Anaconda package) in which the loading, masking, convolution/rebinning,
    and FITS assembly are implemented
  repo: https://github.com/ch-var/SMART.git
- name: Spitzer
  role: Source instrument providing the spectroscopy data of the HERUS ULIRGs that
    are resampled
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:0ec1de77aa3ef3867a0a94fd79ea2f1b468fbd5d8cb7badcefde8d7880260dbb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART.git
  tools:
  - Python 3.10
  - SMART
  - Spitzer
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spitzer-spectra-resolution-matching

## Summary

Preprocesses Spitzer spectroscopy of the 42 HERUS ULIRGs so that each spectrum is convolved/rebinned onto the coarser spectral resolution grid of SMART's pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust), enabling spectroscopy to be included in the Bayesian MCMC SED fitting.

## When to use

Use when adding low/medium-resolution space-based spectra (e.g. Spitzer IRS spectroscopy) to a radiative-transfer SED fit whose model libraries are sampled on a coarser wavelength grid, so spectra and models must be resolution-matched before fitting.

## When NOT to use

- When the data are purely photometric (UV-to-mm fluxes) with no spectroscopy to resample — SMART can be fit on photometry alone without this preprocessing
- When the input spectra are already on the radiative transfer model library's wavelength grid and resolution — rebinning would needlessly degrade them
- When using an energy balance SED-fitting code rather than SMART's radiative transfer model libraries, since the target grid is library-specific

## Inputs

- Spitzer spectroscopy files (wavelength, flux, uncertainty arrays) for each of the 42 HERUS ULIRGs
- Wavelength/resolution grid of SMART's pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust)

## Outputs

- Single multi-extension FITS file with one resolution-matched spectrum extension per source (42 extensions)
- Resolution-matched spectra plus propagated flux uncertainties ready to be ingested by SMART's MCMC SED fitter together with UV-to-mm photometry

## How to apply

Load each Spitzer spectroscopy file for the 42 HERUS ULIRGs in Python 3.10 within the SMART environment and inspect the wavelength, flux, and uncertainty arrays, masking bad or negative-flux channels that would bias the fit. Convolve and rebin each spectrum onto the coarser spectral resolution grid of the pre-computed radiative transfer model libraries, propagating the flux uncertainties through the rebinning so errors remain meaningful in the MCMC likelihood. Assemble the resolution-matched spectra for all 42 galaxies into a single multi-extension FITS file with one extension per source, then run SMART's input reader as a sanity check that the spectra parse correctly and their wavelength grids align with the model library coverage. Only after validation should the resampled spectra be combined with the ultraviolet-to-millimetre photometry as inputs to SMART's Bayesian MCMC fit.

## Related tools

- **SMART** (SED-fitting code whose pre-computed radiative transfer model libraries define the target spectral resolution; its input reader validates the preprocessed spectra) — https://github.com/ch-var/SMART.git
- **Python 3.10** (Language (Anaconda package) in which the loading, masking, convolution/rebinning, and FITS assembly are implemented) — https://github.com/ch-var/SMART.git
- **Spitzer** (Source instrument providing the spectroscopy data of the HERUS ULIRGs that are resampled)

## Examples

```
from astropy.io import fits; hdu = fits.HDUList([fits.PrimaryHDU()] + [fits.ImageHDU(data=rebin(spitzer_spectrum[g])) for g in herus_galaxies]); hdu.writeto('herus_spitzer_resolution_matched.fits')
```

## Evaluation signals

- Output multi-extension FITS file exists and contains exactly 42 spectrum extensions, one per HERUS galaxy
- Every spectrum's wavelength grid is within the radiative transfer model library resolution/coverage; flag any galaxy whose spectrum fails to load
- SMART's input reader parses all preprocessed spectra without errors and confirms grid alignment with the model library wavelength coverage
- Flux uncertainties are propagated through the rebinning (no NaNs or zero-weight channels introduced by masking of bad/negative-flux channels)
- Downstream, SMART's best-fit SED reproduces both the observed photometry and the included Spitzer spectroscopy for each galaxy

## Limitations

- Bad or negative-flux channels must be masked before rebinning or they will bias the fit
- The rebinning degrades the native Spitzer spectral resolution to the (coarser) model library grid; fine spectral features below the library resolution cannot be exploited
- No changelog found for the SMART repository, so version-specific behavior of the preprocessing is not documented

## Evidence

- [abstract] The Spitzer spectroscopy data of the HERUS ULIRGs are included in SMART's fitting after preprocessing to a spectral resolution matched to that of the pre-computed radiative transfer models.: [evidence span withheld — anchor status: not_found]
- [intro] The code is developed in PYTHON 3.10 (Anaconda package).: [evidence span withheld — anchor status: not_found]
- [abstract] SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries, which describe the starburst,: "SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries, which describe the starburst,"
- [readme] To learn how to run SMART and for a helpful user guide, please read the manual.: "To learn how to run <b>SMART</b> and for a helpful user guide, please read the <a href="https://github.com/ch-var/SMART/blob/main/assets/SMART_User_Manual.pdf">manual</a>."
