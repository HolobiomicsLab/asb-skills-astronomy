---
name: fits-spectrum-assembly
description: Use when observational spectra (e.g. Spitzer spectroscopy) must be rebinned/convolved
  onto the coarser wavelength grid of pre-computed radiative transfer model libraries
  and packaged as multi-extension FITS for SED fitting input.
when_to_use_negative:
- When the spectra are already on the radiative transfer model library's resolution
  grid — rebinning/convolving again would degrade the data.
- 'When fitting photometry-only SEDs: this skill concerns including Spitzer spectroscopy
  in SMART''s fitting, not broadband photometric points.'
edam_operation: http://edamontology.org/operation_3223
tools:
- name: SMART
  role: SED fitting code whose input reader validates the assembled multi-extension
    FITS spectra and whose model libraries define the target resolution grid
  repo: https://github.com/ch-var/SMART.git
- name: Python
  role: Environment (Python 3.10, Anaconda) in which the Spitzer spectra are loaded,
    masked, convolved and rebinned
  repo: https://github.com/ch-var/SMART.git
- name: Spitzer
  role: Instrument providing the spectroscopy data of the HERUS ULIRGs that are preprocessed
    and assembled into FITS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:cb25cf91f9b844632e3d42942f4415797333b423e56067855b8f2c87919c8d1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART.git
  tools:
  - Python
  - SMART
  - Spitzer
  edam_operation: http://edamontology.org/operation_3223
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# FITS spectrum assembly

## Summary

Assemble preprocessed, resolution-matched Spitzer spectroscopy of the 42 HERUS ULIRGs into a single multi-extension FITS file (one extension per source) so SMART's radiative-transfer SED fitting can ingest the spectra alongside its pre-computed model libraries.

## When to use

Use when observational spectra (e.g. Spitzer spectroscopy) must be rebinned/convolved onto the coarser wavelength grid of pre-computed radiative transfer model libraries and packaged as multi-extension FITS for SED fitting input.

## When NOT to use

- When the spectra are already on the radiative transfer model library's resolution grid — rebinning/convolving again would degrade the data.
- When fitting photometry-only SEDs: this skill concerns including Spitzer spectroscopy in SMART's fitting, not broadband photometric points.

## Inputs

- Spitzer spectroscopy files (wavelength, flux, uncertainty arrays) for the 42 HERUS ULIRGs
- Wavelength/resolution grid of SMART's pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust)

## Outputs

- Single multi-extension FITS file with one extension per galaxy containing resolution-matched spectra
- Flags/log of galaxies whose spectra fail to load or whose wavelength grids do not match the model library resolution

## How to apply

Load each Spitzer spectroscopy file for the 42 HERUS ULIRGs in Python 3.10 within SMART, inspecting the wavelength, flux and uncertainty arrays and masking bad or negative-flux channels that would bias the fit. Convolve and rebin each spectrum onto the coarser spectral resolution grid of SMART's pre-computed radiative transfer libraries (starburst, AGN torus, host galaxy, polar dust components), propagating flux uncertainties through the rebinning. Write the resolution-matched spectra into a single multi-extension FITS file with one extension per source. Finally, run SMART's input reader as a sanity check that the spectra parse and their grids align with the model library wavelength coverage.

## Related tools

- **SMART** (SED fitting code whose input reader validates the assembled multi-extension FITS spectra and whose model libraries define the target resolution grid) — https://github.com/ch-var/SMART.git
- **Python** (Environment (Python 3.10, Anaconda) in which the Spitzer spectra are loaded, masked, convolved and rebinned) — https://github.com/ch-var/SMART.git
- **Spitzer** (Instrument providing the spectroscopy data of the HERUS ULIRGs that are preprocessed and assembled into FITS)

## Evaluation signals

- Output FITS file exists and contains exactly 42 spectra extensions, one per HERUS ULIRG.
- Every spectrum's wavelength grid is within the radiative transfer model library resolution/coverage.
- SMART's input reader parses the preprocessed spectra without errors (grids align with model library wavelength coverage).
- Bad/negative-flux channels are masked so they do not bias the fit.
- Any galaxy whose spectrum fails to load is explicitly flagged.

## Limitations

- No changelog is available for the SMART repository, so version-specific preprocessing behaviour cannot be verified from the repo.
- Resolution matching is constrained by the pre-computed model libraries; spectra cannot be fit at their native Spitzer resolution.

## Evidence

- [abstract] Spitzer spectra resolution-matched to models: "The Spitzer spectroscopy data of the HERUS ULIRGs are included in the fitting"
- [abstract] HERUS sample size: "test it by fitting the multiwavelength SEDs of the 42 local ultraluminous infrared galaxies (ULIRGs) that constitute the HER"
- [abstract] SMART fits SEDs with four pre-computed radiative transfer libraries: "exclusively with radiative transfer models that currently constitute four types of pre-computed libraries, which describe the starburst"
- [readme] Development environment: "The code is developed in <b>PYTHON 3.10</b> (Anaconda package)."
- [readme] Four library component types incl. AGN torus and polar dust: "active galactic nucleus torus, host galaxy and polar dust components"
