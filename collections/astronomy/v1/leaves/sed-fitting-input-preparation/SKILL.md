---
name: sed-fitting-input-preparation
description: Use when SED fitting must combine spectroscopy (e.g. Spitzer IRS spectra)
  with photometry and radiative transfer model libraries whose wavelength grids are
  coarser than the observed spectra, requiring convolution/rebinning first.
when_to_use_negative:
- Input data are photometric points only — no spectroscopy needs convolution/rebinning
  to the model grid resolution.
- The spectra are already on the same spectral resolution grid as the radiative transfer
  model libraries and parse correctly in SMART's input reader.
- Fitting with non-radiative-transfer SED codes (e.g. energy balance methods) that
  do not use SMART's four pre-computed model libraries.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: SMART
  role: SED fitting code whose radiative transfer model libraries define the target
    spectral resolution; its input reader validates the preprocessed spectra
  repo: https://github.com/ch-var/SMART.git
- name: Python 3.10
  role: Language (Anaconda package) in which the convolution, rebinning, masking,
    and FITS assembly of the Spitzer spectra are performed
- name: Spitzer
  role: Source of the HERUS ULIRG spectroscopy data being preprocessed into the fitting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:867969050e187085db5c16929502fc8df4342ef92a742626861880e8e31bb061
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART.git
  tools:
  - Python 3.10
  - SMART
  - Spitzer
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sed-fitting-input-preparation

## Summary

Preprocess observational spectroscopy (e.g. Spitzer spectra of the 42 HERUS ULIRGs) to the spectral resolution of SMART's pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust) before Bayesian MCMC SED fitting, assembling resolution-matched spectra into a multi-extension FITS file.

## When to use

Use when SED fitting must combine spectroscopy (e.g. Spitzer IRS spectra) with photometry and radiative transfer model libraries whose wavelength grids are coarser than the observed spectra, requiring convolution/rebinning first.

## When NOT to use

- Input data are photometric points only — no spectroscopy needs convolution/rebinning to the model grid resolution.
- The spectra are already on the same spectral resolution grid as the radiative transfer model libraries and parse correctly in SMART's input reader.
- Fitting with non-radiative-transfer SED codes (e.g. energy balance methods) that do not use SMART's four pre-computed model libraries.

## Inputs

- Spitzer spectroscopy files (wavelength, flux, uncertainty arrays) for each of the 42 HERUS ULIRGs
- Pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust) with their target spectral resolution grids

## Outputs

- Resolution-matched Spitzer spectra assembled into a single multi-extension FITS file with one extension per galaxy
- SMART-parsable spectra whose wavelength grids align with the radiative transfer model library coverage

## How to apply

Load the Spitzer spectroscopy files for each galaxy in Python 3.10 within the SMART environment, and mask bad or negative-flux channels that would bias the fit. Convolve and rebin each spectrum onto the coarser spectral resolution grid of the pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust components), propagating flux uncertainties through the rebinning. Assemble the resolution-matched spectra for all 42 HERUS ULIRGs into a single multi-extension FITS file with one extension per source. Finally, run SMART's input reader as a sanity check that the preprocessed spectra parse correctly and their wavelength grids align with the model library wavelength coverage before the MCMC fitting.

## Related tools

- **SMART** (SED fitting code whose radiative transfer model libraries define the target spectral resolution; its input reader validates the preprocessed spectra) — https://github.com/ch-var/SMART.git
- **Python 3.10** (Language (Anaconda package) in which the convolution, rebinning, masking, and FITS assembly of the Spitzer spectra are performed)
- **Spitzer** (Source of the HERUS ULIRG spectroscopy data being preprocessed into the fitting)

## Evaluation signals

- The output FITS file exists and contains exactly 42 spectra extensions (one per HERUS ULIRG).
- Every spectrum's wavelength grid is within the radiative transfer model library resolution.
- Flag (or confirm absence of) any galaxy whose spectrum fails to load in SMART's input reader.
- Bad/negative-flux channels are masked and flux uncertainties are propagated through the rebinning (no negative-flux channels survive into the fitting input).

## Limitations

- The rebinning must match the resolution of SMART's currently available pre-computed libraries (starburst, AGN torus, host galaxy, polar dust); changes to the model libraries would require re-matching the spectra.
- Bad or negative-flux channels must be masked, otherwise the fit is biased.
- No changelog found for the repository, so preprocessing compatibility across SMART versions is not documented.

## Evidence

- [abstract] The Spitzer spectroscopy data of the HERUS ULIRGs are included in SMART's fitting after preprocessing to a spectral resolution matched to that of the pre-computed radiative transfer models.: "The Spitzer spectroscopy data of the HERUS ULIRGs are included in the fitting"
- [abstract] SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries, which describe the starburst,: "SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries, which describe the starburst,"
- [intro] The code is developed in PYTHON 3.10 (Anaconda package).: [evidence span withheld — anchor status: not_found]
- [readme] To learn how to run SMART and for a helpful user guide, please read the manual.: "To learn how to run <b>SMART</b> and for a helpful user guide, please read the"
