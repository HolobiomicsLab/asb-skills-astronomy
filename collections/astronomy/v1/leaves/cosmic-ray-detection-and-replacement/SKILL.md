---
name: cosmic-ray-detection-and-replacement
description: Use when input is an AAOmega-format (or Marz-format) FITS spectrum with
  flux and variance extensions that needs cleaning — NaN/negative/over-threshold flux
  pixels, negative-variance pixels, or >30 sigma cosmic-ray spikes — prior to redshift
  cross-correlation.
when_to_use_negative:
- The spectrum is already clean and continuum-removed and only needs redshift matching
  — go straight to Marz's automatic matching (./marz.sh) or manual redshifting
- The input is not a 1D fibre spectrum with paired flux and variance extensions (e.g.
  2D images or feature tables), where the second-neighbour pixel test and polynomial
  continuum model do not apply
- Cosmic rays have been rejected upstream by the instrument pipeline and the task
  is only redshift classification (e.g. QOP merging or template comparison)
tools:
- name: Marz
  role: Defines the bad-pixel flagging rule, the second-neighbour >30 sigma cosmic-ray
    test with 9/19-pixel replacement windows, and the iterative 6-degree rejected
    continuum polynomial used in the AAOmega preprocessing stage
  repo: https://github.com/samreay/Marz
- name: astropy.io.fits
  role: Python library used to load the AAOmega FITS flux, variance, and wavelength
    extensions and to write the preprocessed spectrum back to FITS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.03.001
    title: 'Marz: Manual and automatic redshifting software'
  source_content_hash: sha256:610b553c9bed9113a9178a05a8428f3ddb106f48efa03944504a34a17f3acb4a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/samreay/Marz
  tools:
  - Marz
  - astropy.io.fits
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# cosmic-ray-detection-and-replacement

## Summary

Reproduces Marz's preprocessing stage for AAOmega spectra: flag bad pixels (NaN, negative, over-threshold intensity, or negative variance), detect cosmic rays with a second-neighbour test at >30 sigma, replace them using 9-pixel and 19-pixel windows, and remove the continuum with an iterative 6-degree rejected polynomial before automatic redshift matching.

## When to use

Use when input is an AAOmega-format (or Marz-format) FITS spectrum with flux and variance extensions that needs cleaning — NaN/negative/over-threshold flux pixels, negative-variance pixels, or >30 sigma cosmic-ray spikes — prior to redshift cross-correlation.

## When NOT to use

- The spectrum is already clean and continuum-removed and only needs redshift matching — go straight to Marz's automatic matching (./marz.sh) or manual redshifting
- The input is not a 1D fibre spectrum with paired flux and variance extensions (e.g. 2D images or feature tables), where the second-neighbour pixel test and polynomial continuum model do not apply
- Cosmic rays have been rejected upstream by the instrument pipeline and the task is only redshift classification (e.g. QOP merging or template comparison)

## Inputs

- AAOmega-format FITS spectrum file with intensity/flux, variance, and wavelength extensions (e.g. an example spectrum from the Marz repository)
- Configurable bad-pixel intensity threshold for the over-threshold test

## Outputs

- Cleaned flux array with bad pixels masked and cosmic-ray spikes replaced
- Cleaned variance array with negative-variance pixels handled
- Continuum-subtracted flux (residual after 6-degree rejected polynomial subtraction)
- Preprocessed output FITS file (e.g. preprocessed_spectrum.fits)

## How to apply

Load the FITS file's flux, variance, and wavelength extensions (e.g. with astropy.io.fits). First apply the bad-pixel rule from Marz: mark a pixel as bad if the intensity is NaN, negative, or above a configurable threshold, or if its variance is negative. Then run the cosmic-ray detection: flag pixels that pass the second-neighbour test at >30 sigma and replace flagged values using the 9-pixel and 19-pixel replacement windows. Finally fit the continuum with an iterative 6-degree rejected polynomial (3.5 sigma rejection, up to 15 iterations) and subtract the fit from the flux, writing the cleaned flux and variance to a new FITS file. The rationale is that the modified autoz cross-correlation matcher in Marz performs best on spectra free of spurious spikes, masked detector defects, and residual continuum curvature.

## Related tools

- **Marz** (Defines the bad-pixel flagging rule, the second-neighbour >30 sigma cosmic-ray test with 9/19-pixel replacement windows, and the iterative 6-degree rejected continuum polynomial used in the AAOmega preprocessing stage) — https://github.com/samreay/Marz
- **astropy.io.fits** (Python library used to load the AAOmega FITS flux, variance, and wavelength extensions and to write the preprocessed spectrum back to FITS)

## Examples

```
python -c "from astropy.io import fits; f=fits.open('example_spectrum.fits'); flux,var,wave=f[0].data,f[1].data,f[2].data; bad=(~__import__('numpy').isfinite(flux))|(flux<0)|(var<0)"
```

## Evaluation signals

- Output FITS file (preprocessed_spectrum.fits) exists and is readable with astropy.io.fits
- No NaN, negative, or over-threshold intensity pixels and no negative-variance pixels remain unmasked in the cleaned flux/variance arrays
- Residual continuum-subtracted flux shows no remaining >30 sigma cosmic-ray spikes on visual and numerical inspection
- Masked-pixel map, replaced spike locations, and flattened continuum match expectations when plotted against the raw input spectrum

## Limitations

- The bad-pixel intensity threshold is configurable, so results depend on the chosen value for the instrument/dataset
- Marz accepts generic FITS files, but the input must follow a well-defined format (flux/variance/wavelength extensions) or the preprocessing cannot proceed
- Marz was developed as a Javascript web-application without significant supporting scientific libraries, and mathematical library support for Javascript was still in its infancy at the time, so Python reimplementation must faithfully mirror Marz's parameters

## Evidence

- [other] Second-neighbour test at >30 sigma with 9/19-pixel replacement: [evidence span withheld — anchor status: not_found]
- [other] Bad-pixel definition rule: [evidence span withheld — anchor status: not_found]
- [other] Continuum subtraction parameters: [evidence span withheld — anchor status: not_found]
- [readme] AAOmega input context from README: "This project aims to take spectra from the AAOmega spectrograph and redshift them via an online application, for application in cosmology."
- [abstract] Matching stage the preprocessing feeds into: "a modified version of the autoz cross-correlation algorithm is used to match input spectra against a variety of stellar and galaxy templates"
