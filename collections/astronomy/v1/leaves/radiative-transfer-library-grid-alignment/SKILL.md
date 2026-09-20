---
name: radiative-transfer-library-grid-alignment
description: Use when mid-IR spectroscopy (e.g. Spitzer IRS spectra) must be included
  in an SED fit that is driven by pre-computed model libraries whose spectral resolution
  is coarser than the data, requiring convolution/rebinning with uncertainty propagation
  before fitting.
when_to_use_negative:
- The analysis uses only broadband photometry with no mid-IR spectroscopy, so no resolution
  matching is needed before the SMART fit
- The spectroscopic data are already provided on the radiative transfer model library's
  wavelength grid and uncertainties are pre-propagated
- The goal is fitting SEDs with energy-balance methods rather than SMART's radiative-transfer-only
  libraries, since those tools handle spectra differently
tools:
- name: SMART
  role: SED fitting code whose pre-computed radiative transfer model libraries define
    the target spectral-resolution grid; its input reader validates the preprocessed
    spectra
  repo: https://github.com/ch-var/SMART.git
- name: Python 3.10
  role: Environment (Anaconda package) in which the Spitzer spectra are loaded, masked,
    convolved, and rebinned
  repo: https://github.com/ch-var/SMART.git
- name: Spitzer
  role: Source of the mid-IR spectroscopy data of the HERUS ULIRGs being preprocessed
    for the fit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:a196d8f3d104f221cf23d36621d81437c6e61fb724a17ccc545d2fc17512e204
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
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# radiative-transfer-library-grid-alignment

## Summary

Preprocesses Spitzer spectroscopy (wavelength, flux, uncertainty arrays) of the 42 HERUS ULIRGs onto the coarser spectral-resolution grid of SMART's pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust) so mid-IR spectra can enter the Bayesian MCMC SED fit alongside photometry.

## When to use

Use when mid-IR spectroscopy (e.g. Spitzer IRS spectra) must be included in an SED fit that is driven by pre-computed model libraries whose spectral resolution is coarser than the data, requiring convolution/rebinning with uncertainty propagation before fitting.

## When NOT to use

- The analysis uses only broadband photometry with no mid-IR spectroscopy, so no resolution matching is needed before the SMART fit
- The spectroscopic data are already provided on the radiative transfer model library's wavelength grid and uncertainties are pre-propagated
- The goal is fitting SEDs with energy-balance methods rather than SMART's radiative-transfer-only libraries, since those tools handle spectra differently

## Inputs

- Spitzer spectroscopy files (wavelength, flux, uncertainty arrays) for the 42 HERUS ULIRGs
- Wavelength grids / spectral resolution of the pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust)

## Outputs

- Resolution-matched, quality-masked Spitzer spectra rebinning-consistent with the radiative transfer model grid
- Single multi-extension FITS file with one extension per HERUS ULIRG containing the preprocessed spectra

## How to apply

Load each of the 42 HERUS ULIRG Spitzer spectroscopy files in Python 3.10 within the SMART environment and inspect the wavelength, flux, and uncertainty arrays, masking bad or negative-flux channels that would bias the fit. Convolve and rebin each spectrum onto the coarser wavelength grid of the radiative transfer libraries (starburst, AGN torus, host galaxy, polar dust components), propagating flux uncertainties through the rebinning so the Bayesian MCMC weights the data correctly. Assemble the resolution-matched spectra for all 42 galaxies into a single multi-extension FITS file with one extension per source. Finally, run SMART's input reader as a sanity check that the preprocessed spectra parse correctly and their grids align with the model library wavelength coverage.

## Related tools

- **SMART** (SED fitting code whose pre-computed radiative transfer model libraries define the target spectral-resolution grid; its input reader validates the preprocessed spectra) — https://github.com/ch-var/SMART.git
- **Python 3.10** (Environment (Anaconda package) in which the Spitzer spectra are loaded, masked, convolved, and rebinned) — https://github.com/ch-var/SMART.git
- **Spitzer** (Source of the mid-IR spectroscopy data of the HERUS ULIRGs being preprocessed for the fit)

## Examples

```
python preprocess_spitzer.py --input spitzer/ --galaxies 42 --grid rt_model_library.fits --output herus_spitzer_matched.fits
```

## Evaluation signals

- The output FITS file exists and contains exactly 42 spectra extensions, one per HERUS ULIRG
- Every spectrum's wavelength grid lies within (is coarser than or equal to) the radiative transfer model library resolution
- SMART's input reader parses the preprocessed spectra without errors and their grids align with the model library wavelength coverage
- No unmasked bad/negative-flux channels remain in the spectra fed to the fit
- Galaxies whose spectra fail to load are flagged rather than silently dropped

## Limitations

- Bad or negative-flux channels in the Spitzer spectra must be masked manually or they bias the fit
- The radiative transfer libraries are pre-computed with fixed coverage, so spectra must be degraded to their coarser resolution — no finer model grid is available
- SMART is developed in PYTHON 3.10 (Anaconda package); other environments are not described by the article
- No changelog is available for the repository, so version-specific behavior of the preprocessing is undocumented

## Evidence

- [abstract] The Spitzer spectroscopy data of the HERUS ULIRGs are included in the fitting: "The Spitzer spectroscopy data of the HERUS ULIRGs are included in the fitting"
- [abstract] test it by fitting the multiwavelength SEDs of the 42 local ultraluminous infrared galaxies (ULIRGs) that constitute the HERschel Ultraluminous Infrared Galaxy Survey (HERUS) sample: "test it by fitting the multiwavelength SEDs of the 42 local ultraluminous infrared galaxies (ULIRGs) that constitute the HERschel Ultraluminous Infrared Galaxy Survey (HERUS) sample"
- [readme] SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries, which describe the starburst,: "SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries, which describe the starburst,"
- [intro] The code is developed in PYTHON 3.10 (Anaconda package).: [evidence span withheld — anchor status: not_found]
